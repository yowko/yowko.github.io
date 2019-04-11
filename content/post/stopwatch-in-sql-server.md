---
title: "如何得知 script 執行時間 - Stopwatch in SQL Server ？！"
date: 2017-06-30T23:53:00+08:00
lastmod: 2018-09-23T23:53:40+08:00
draft: false
tags: ["SQL Server"]
slug: "stopwatch-in-sql-server"
aliases:
    - /2017/06/stopwatch-in-sql-server.html
---
# 如何得知 script 執行時間 - Stopwatch in SQL Server ？！
SQL script 執行緩慢偶爾就會出現在討論的話題中，script 優化也是 db 效能調校最直接有效跟成本效益比最高的方式。

只是面對重輒幾百上千行的 SQL Script，想要精準地找到哪段語法效率不佳這件事就很困難了，我的做法會先從執行時間較久的語法開頭，看有沒有機會優化跟調整，所以今天分享一下如何衡量 SQL Script 的執行時間

## 一、使用統計資料

1.  語法

    ```sql
    SET STATISTICS TIME { ON | OFF }
    ```

2.  範例

    ```sql
    USE AdventureWorks2014;  
    GO         
    SET STATISTICS TIME ON;  
    GO  
    SELECT ProductID, StartDate, EndDate, StandardCost   
    FROM Production.ProductCostHistory  
    WHERE StandardCost < 500.00;  
    GO  
    SET STATISTICS TIME OFF;  
    GO
    ```

3.  結果

    ```
    SQL Server parse and compile time: 
    CPU time = 0 ms, elapsed time = 5 ms.
    SQL Server parse and compile time: 
    CPU time = 0 ms, elapsed time = 0 ms.
                (269 row(s) affected)
                SQL Server Execution Times:
    CPU time = 0 ms,  elapsed time = 0 ms.
    SQL Server parse and compile time: 
    CPU time = 0 ms, elapsed time = 0 ms.
    ```
    
    ![1statistics](https://user-images.githubusercontent.com/3851540/27743430-909c6a56-5dee-11e7-9f66-d9c77caee8bd.png)

## 二、啟用 `Include Client Statistics`

1.  SSMS 主選單 Query --> Include Client Statistics

    ![2includeclientstat](https://user-images.githubusercontent.com/3851540/27743431-909ce31e-5dee-11e7-9ddb-f4c7b5433168.png)

2.  範例

    ```sql
    USE AdventureWorks2014;  
    GO         
    SELECT ProductID, StartDate, EndDate, StandardCost   
    FROM Production.ProductCostHistory  
    WHERE StandardCost > 500.00;  
    GO
    ```

3.  結果

    ![2includeresult](https://user-images.githubusercontent.com/3851540/27743432-90a251aa-5dee-11e7-857c-6ac4bdd98084.png)

## 三、手刻計時器

> 上面的兩種方法都比較適合用來衡量單一 script 執行時間，如果想要得知單一 script 中的不同 statement 執行時間，這個方法最適合

1.  語法

    ```SQL
    DECLARE @t1 DATETIME;
    DECLARE @t2 DATETIME;
    
    SET @t1 = GETDATE();
    SELECT /* query one */ 1 ;
    SET @t2 = GETDATE();
    SELECT DATEDIFF(millisecond,@t1,@t2) AS elapsed_ms;
    
    SET @t1 = GETDATE();
    SELECT /* query two */ 2 ;
    SET @t2 = GETDATE();
    SELECT DATEDIFF(millisecond,@t1,@t2) AS elapsed_ms;
    ```

2.  範例

    ```sql
    USE AdventureWorks2014;  
    GO         
    DECLARE @t1 DATETIME;
    DECLARE @t2 DATETIME;
    SET @t1 = GETDATE();
    SELECT count(1)
    FROM [Sales].[SalesPerson]
    SET @t2 = GETDATE();
    SELECT DATEDIFF(millisecond,@t1,@t2) AS elapsed_ms;
    
    SET @t1 = GETDATE();
    SELECT count(1)
    FROM [Sales].[Store]
    SET @t2 = GETDATE();
    SELECT DATEDIFF(millisecond,@t1,@t2) AS elapsed_ms;
    GO
    ```

3.  結果

    ![3inlineresult](https://user-images.githubusercontent.com/3851540/27743433-90a5e450-5dee-11e7-9f5c-cbe4612627ad.png)

## 心得

雖然知道 script 執行時間並沒有直接效益，但透過簡單的方式先定位出 script 可能的執行瓶頸，再針對可能的效能瓶頸做進一步優化，還是比一行一行 SQL script 逐一檢查來得有效率

# 參考資訊

1.  [SET STATISTICS TIME (Transact-SQL)](https://docs.microsoft.com/ZH-tw/sql/t-sql/statements/set-statistics-time-transact-sql)
2.  [Show Query Execution Time](http://www.sqlserver.info/management-studio/show-query-execution-time/)
