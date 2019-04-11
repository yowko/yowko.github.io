---
title: "SQL Server 如何紀錄執行錯誤"
date: 2017-04-25T02:07:00+08:00
lastmod: 2018-09-185T02:07:18+08:00
draft: false
tags: ["Debug","SQL Server"]
slug: "sql-server-log-exception"
aliases:
    - /2017/04/sql-server-log-exception.html
---
# SQL Server 如何紀錄執行錯誤
程式在 production 環境上出現錯誤，大家會怎麼 debug 呢？最傳統也最普遍的做法當然就是追查各式各樣的 log，現在發生錯誤的對象換做是 db 該怎麼辦呢？ 如果是 DML(Data Manipulation Language：insert、update、delete) 或是 DQL(Data Query Language：select) 相關指令在執行發生錯誤會直接回傳給 AP 接著就會被 AP log 起來，相對容易除錯，一旦錯誤是發生在 stored procedure 裡難度就增加很多，一般來說 stored procedure 常常有一定程度的邏輯存在，以前我總是透過逐行下 log 的方式來除錯，只要 log 沒下好就得重來，加上系統龐大 stored procedure 眾多，debug 的慘況可想而知，所以我們一定得透過更有效率的方式來處理

主要靈感來源是 [AdventureWorks2014](https://msftdbprodsamples.codeplex.com/downloads/get/880661)，其中 ErrorLog 是用來紀錄錯誤資訊的 table，uspLogError 則是用來寫入錯誤資訊的 stored procedure，後來參考 [Exception Handling in SQL Server](https://www.codeproject.com/Articles/38211/Exception-Handling-in-SQL-Server) 加入 host 欄位

## 建立儲存錯誤資訊的資料表

```SQL
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

SET ANSI_PADDING ON
GO

CREATE TABLE [dbo].[ErrorLogs](
 [Id] [int] IDENTITY(1,1) NOT NULL,
 [ErrorNo] [int] NOT NULL,
 [ErrorState] [int] NULL,
 [ErrorSeverity] [int] NULL,
 [ErrorLine] [int] NULL,
 [ErrorProc] [varchar](200) NULL,
 [ErrorMSG] [varchar](max) NOT NULL,
 [UserName] [varchar](200) NOT NULL,
 [HostName] [varchar](200) NULL,
 [ErrorDate] [datetime] NOT NULL,
 CONSTRAINT [PK_ErrorMSG] PRIMARY KEY CLUSTERED 
(
 [Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]

GO

SET ANSI_PADDING ON
GO

ALTER TABLE [dbo].[ErrorLogs] ADD  CONSTRAINT [DF_ErrorMSG_ErrorDate]  DEFAULT (getdate()) FOR [ErrorDate]
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'錯誤代碼' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorNo'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'錯誤狀態碼' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorState'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'錯誤嚴重性' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorSeverity'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'錯誤行號' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorLine'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'發生錯誤的程序名稱' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorProc'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'錯誤訊息' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorMSG'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'執行錯誤程序的 username' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'UserName'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'執行錯誤程序的 hostname' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'HostName'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'錯誤發生時間' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'ErrorLogs', @level2type=N'COLUMN',@level2name=N'ErrorDate'
GO
```

![1errorlogs](https://cloud.githubusercontent.com/assets/3851540/25351207/8a683a02-295a-11e7-8de1-bf3140cabd09.png)

## 建立寫入錯誤資訊的 stored procedure

*   uspLogErrors

    ```sql
    SET ANSI_NULLS ON
    GO

    SET QUOTED_IDENTIFIER ON
    GO


    CREATE PROCEDURE [dbo].[uspLogErrors] 
        @ErrorLogID [int] = 0 OUTPUT 
    AS                               
    BEGIN
        SET NOCOUNT ON;

        SET @ErrorLogID = 0;

        BEGIN TRY
            IF ERROR_NUMBER() IS NULL
                RETURN;

            IF XACT_STATE() = -1
            BEGIN
                PRINT '因為仍有未 commit 的 transaction 所以無法紀錄錯誤,為了成功紀錄錯誤訊息請在執行 uspLogErrors 前先 rollback transaction';
                RETURN;
            END

            INSERT [dbo].[ErrorLogs] 
                (
    ErrorNo,
    ErrorState,
    ErrorSeverity,
    ErrorLine,
    ErrorProc,
    ErrorMSG,
    UserName,
    HostName
                ) 
            VALUES 
                (
                ERROR_NUMBER(),
    ERROR_STATE(),
                ERROR_SEVERITY(),
                ERROR_LINE(),
                ERROR_PROCEDURE(),
                ERROR_MESSAGE(),
    CURRENT_USER,
    Host_NAME()  
                );

            SET @ErrorLogID = @@IDENTITY;
        END TRY
        BEGIN CATCH
            PRINT '執行 uspLogErrors 發生錯誤: ';
            EXECUTE [dbo].[uspPrintError];
            RETURN -1;
        END CATCH
    END;

    GO

    EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'將 TRY CATCH 中所 catch 到錯誤資訊紀錄至 ErrorLogs ' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'PROCEDURE',@level1name=N'uspLogErrors'
    GO

    EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'回傳寫入至 ErrorLogs 的 id' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'PROCEDURE',@level1name=N'uspLogErrors', @level2type=N'PARAMETER',@level2name=N'@ErrorLogID'
    GO
    ```
    
    > 為了避免用來紀錄錯誤訊息的 stored procedure 也出現錯誤，另外加上一支只顯示錯誤資訊的 stored procedure

*   uspPrintError

    ```sql
    SET ANSI_NULLS ON
    GO
                SET QUOTED_IDENTIFIER ON
    GO
                CREATE PROCEDURE [dbo].[uspPrintError] 
    AS
    BEGIN
        SET NOCOUNT ON;
                    PRINT 'Error ' + CONVERT(varchar(50), ERROR_NUMBER()) +
            ', Severity ' + CONVERT(varchar(5), ERROR_SEVERITY()) +
            ', State ' + CONVERT(varchar(5), ERROR_STATE()) + 
            ', Procedure ' + ISNULL(ERROR_PROCEDURE(), '-') + 
            ', Line ' + CONVERT(varchar(5), ERROR_LINE());
        PRINT ERROR_MESSAGE();
    END;
    GO
    EXEC sys.sp_addextendedproperty @name=N'MS_Description', value=N'Prints error information about the error that caused xecution to jump to the CATCH block of a TRY...CATCH construct. hould be executed from within the scope of a CATCH block otherwise t will return without printing any error information.' , level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'PROCEDURE',level1name=N'uspPrintError'
    GO
    ```

## 執行 script 並加入錯誤處理機制

*   script

    ```sql
    BEGIN TRY
        SELECT 1+Name from [Sales].[Currency]
    END TRY
    BEGIN CATCH
        EXECUTE [dbo].[uspLogErrors];
    END CATCH;
    ```
*   stored procedure

    ```sql
    CREATE PROCEDURE [dbo].[uspSelectSalesCurrency]
    AS
    BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        SELECT 1+Name from [Sales].[Currency]
    END TRY
        BEGIN CATCH
            EXECUTE [dbo].[uspLogErrors];
        END CATCH;
    END
    GO
    ```
    *   執行
        
        > `exec uspSelectSalesCurrency`

## 實際效果

![2result](https://cloud.githubusercontent.com/assets/3851540/25351208/8a6be67a-295a-11e7-82fc-c07d5cd6854c.png)

*   ErrorProc 表示發生錯誤的程序

    *   `NULL` 即是直接執行 script



# 參考資訊

1.  [AdventureWorks2014](https://msftdbprodsamples.codeplex.com/downloads/get/880661)
2.  [Exception Handling in SQL Server](https://www.codeproject.com/Articles/38211/Exception-Handling-in-SQL-Server)
3.  [TRY...CATCH (Transact-SQL)](https://msdn.microsoft.com/zh-tw/library/ms175976.aspx)
