---
title: "相同的 SQL script 在 Stored Procedure 裡跑比較慢？！"
date: 2017-04-23T02:57:00+08:00
lastmod: 2018-09-18T02:57:03+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-script-parameter-sniffing"
aliases:
    - /2017/04/sql-script-stored-procedure.html
---
# 相同的 SQL script 在 Stored Procedure 裡跑比較慢？！
你遇過把原本執行效率不錯的 sql script 搬到 stored procedure 後執行效率大降的情形嗎？我遇到一次：問題發生原因是 parameter sniffing (參數探測)，至於可不可以把所有 script 移到 sp 後變慢的問題都歸咎給 parameter sniffing (參數探測) 這就得請教專業的 DBA 了，今天就分享之前遇到的案例及解決方法

## 基本設定

1.  建立測試資料表

    ```sql
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    SET ANSI_PADDING ON
    GO
    CREATE TABLE [dbo].[TestParameterSniffing](
        [Id] [int] IDENTITY(1,1) NOT NULL,
        [Name] [varchar](50) NOT NULL,
        [Birthday] [datetime] NOT NULL,
        CONSTRAINT [PK_TestParameterSniffing] PRIMARY KEY CLUSTERED 
        (
        [Id] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
        ) ON [PRIMARY]
    GO
    SET ANSI_PADDING OFF
    GO
    ```

2.  建立 index

    ```sql
    CREATE NONCLUSTERED INDEX [NonClusteredIndex-birthday] ON [dbo].[TestParameterSniffing]
    (
    [Birthday] ASC
    )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)
    Go
    ```
3.  建立測試資料

    ```sql
    declare  @i  int ,  @birthAdd  int ; 
    set  @i  =  0 ; 
    while  @i  <  1000000 
    begin 
    set @i  =  @i  +  1 ; 
    set @birthAdd=cast (rand ()*3650 as int); 
        insert  TestParameterSniffing  ( Birthday ,  Name ) 
        values  ( dateadd ( dd ,  @birthAdd ,  cast ( '2010/01/01'  as  datetime )),  'yowko_'+cast(@i as varchar(10)) ); 
    end
    ```
## 問題模擬

1.  Literal 寫法(將條件直接寫在 script 中)
    *   script

        ```sql
        select  Birthday ,  Name  from  TestParameterSniffing 
        where  Birthday  between  '2015-01-01'  and  '2015-12-31'
        ```

    *   執行計劃

        ![1literal](https://cloud.githubusercontent.com/assets/3851540/25307216/a0f8fc34-27cf-11e7-863b-fe12ee20b68c.png)

2.  將 script 移入 stored procedure 中

    ```sql
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    create  procedure  [dbo].[uspParamaterSniffing] 
               @BeginDate  datetime , 
               @EndDate  datetime 
    as 
    set  nocount  on ; 
    select  Birthday ,  Name  from  TestParameterSniffing 
    where  Birthday  between  @BeginDate  and  @EndDate        
    GO
    ```

3.  使用不同參數執行 sp

    *   正確執行計劃
        *   script
            
            ```sql
            dbcc  freeproccache ; --清除執行計劃
            exec  uspParamaterSniffing  '2011-01-01' ,  '2011-12-31' ; 
            exec  uspParamaterSniffing  '2015-01-01' ,  '2015-01-03' ;
            ```

        *   執行計劃
            
            ![2sp1](https://cloud.githubusercontent.com/assets/3851540/25307215/a0f7b5e0-27cf-11e7-8960-8eddf6bcf7c8.png)

    *   錯誤執行計劃

        *   script

            ```sql
            dbcc  freeproccache ; --清除執行計劃
            exec  uspParamaterSniffing  '2015-01-01' ,  '2015-01-03' ;
            exec  uspParamaterSniffing  '2011-01-01' ,  '2011-12-31' ;
            ```
        *   執行計劃
            
            ![3sp2](https://cloud.githubusercontent.com/assets/3851540/25307214/a0f70776-27cf-11e7-8e8b-d9f44567946a.png)

## 如何解決

原本預期走 index scan 的查詢竟然出現 index + key look，這就是 parameter sniffing (參數探測) 造成執行計劃誤用，針對這個案例可以透過使用 local variable 來接外部傳入的參數改善

1.  修改 sp

    ```sql
    ALTER  procedure  [dbo].[uspParamaterSniffing] 
    @BeginDate  datetime , 
    @EndDate  datetime 
    as 
    set  nocount  on ; 
    declare  @_BeginDate  datetime ,  @_EndDate  datetime ; 
    set  @_BeginDate  =  @BeginDate ; 
    set  @_EndDate  =  @EndDate ;            
    select  Birthday ,  Name  from  TestParameterSniffing 
    where  Birthday  between  @_BeginDate  and  @_EndDate
    ```
2.  誤用執行計畫的問題已解決

    *   script 1

        ```sql
        dbcc  freeproccache ; --清除執行計劃
        exec  uspParamaterSniffing  '2011-01-01' ,  '2011-12-31' ; 
        exec  uspParamaterSniffing  '2015-01-01' ,  '2015-01-03' ;
        ```
    *   執行計劃 1

        ![4sp3](https://cloud.githubusercontent.com/assets/3851540/25307217/a0fa9562-27cf-11e7-9510-54c1ab48b475.png)

    *   script 2

        ```sql
        dbcc  freeproccache ; --清除執行計劃
        exec  uspParamaterSniffing  '2015-01-01' ,  '2015-01-03' ;
        exec  uspParamaterSniffing  '2011-01-01' ,  '2011-12-31' ;
        ```
    *   執行計劃 2

        ![5sp4](https://cloud.githubusercontent.com/assets/3851540/25307218/a0fbe7c8-27cf-11e7-8f41-9b5fab6f8613.png)

## 心得

感謝前同事強大 dba 的指導，讓 SQL Server 相關知識及功力能再加強些，也順利避免遇到類似問題求救無門

# 參考資訊
1.  [SQL筆記：Literal, Variable與Parameter](http://blog.darkthread.net/post-2015-08-16-literal-var-param-and-exec-plan.aspx)
