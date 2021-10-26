---
title: "試著看懂 SQL Server IO 統計資訊"
date: 2016-12-13T00:42:34+08:00
lastmod: 2021-10-26T00:42:34+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-io-statistics"
aliases:
    - /2016/12/sql-server-io.html
---

## 試著學會看懂 SQL Server IO 統計資訊

SQL Server IO 統計資訊是重要的效能調校依據，第一步就是嘗試看懂其中的涵意。

1. `SET STATISTICS IO { ON | OFF }`

    >用來開啟 IO 統計資訊

2. `SET STATISTICS IO` 設定動作是發生在 `執行階段` (execute or run time)，而不是在 `剖析階段` (parse time)。
3. 當 Transact-SQL 取得 LOB 資料行時，可能需要往返 LOB 樹狀結構多次。 而造成 SET STATISTICS IO 報告的數字高於預期的邏輯讀取次數。

## scan (掃瞄計數)

為了建構輸出的最終資料集，在達到分葉層級之後朝任何方向啟動以擷取所有值的搜尋/掃瞄次數。

>以下範例使用 **`微軟 [AdventureWorks2014].[Sales].[Customer]`**

1. 如果使用的索引是`主索引鍵的叢集索引`或`唯一索引`，而且您只要搜尋一個值，掃瞄計數就是 0。 例如 WHERE Primary_Key_Column = `value`。
    - 1-1. 主索引鍵的叢集索引(clustered index on a primary key )

        ```sql
        SELECT  
        [CustomerID]
            ,[PersonID]
            ,[StoreID]
            ,[TerritoryID]
            ,[AccountNumber]
            ,[rowguid]
            ,[ModifiedDate]
        FROM [AdventureWorks2014].[Sales].[Customer]
        where [CustomerID]=1
        ```

        ![CustomerID](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580432e4655468d3c9285abc/c02dfbdab118ef32a265ed0554bedafd/CustomerID_%E7%BB%93%E6%9E%9C.png)

    - 1-2. 唯一索引(unique index)

        ```sql
        SELECT  
        [CustomerID]
            ,[PersonID]
            ,[StoreID]
            ,[TerritoryID]
            ,[AccountNumber]
            ,[rowguid]
            ,[ModifiedDate]
        FROM [AdventureWorks2014].[Sales].[Customer]
        where [AccountNumber]='AW00000001'
        ```

        ![unique](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580432e4655468d3c9285abc/e5985a6bd315998caa6efa0c93750b18/unique_%E7%BB%93%E6%9E%9C.png)

2. 當您要使用在非主索引鍵資料行上定義的非唯一叢集索引來搜尋一個值時，掃瞄計數就是 1。 進行這項作業是為了檢查您所搜尋的索引鍵值是否有重複的值。 例如 WHERE Clustered_Index_Key_Column = `value`。  

    >實際測試下 ，有下列三種情形會出現 掃瞄計數=`1`

    - 2-1. 全部掃瞄  

        ```sql
        SELECT  
        [CustomerID]
            ,[PersonID]
            ,[StoreID]
            ,[TerritoryID]
            ,[AccountNumber]
            ,[rowguid]
            ,[ModifiedDate]
        FROM [AdventureWorks2014].[Sales].[Customer]
        ```

        ![all](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580432e4655468d3c9285abc/e24ae4a07e83004ab9f34ff7dab58d30/all_%E7%BB%93%E6%9E%9C.png)

    - 2-2. 非唯一、非叢集索引(non-unique non-clustered index)

        ```sql
        SELECT  
        [CustomerID]
            ,[PersonID]
            ,[StoreID]
            ,[TerritoryID]
            ,[AccountNumber]
            ,[rowguid]
            ,[ModifiedDate]
        FROM [AdventureWorks2014].[Sales].[Customer]
        where [TerritoryID]=1
        ```

        ![non -unique](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580432e4655468d3c9285abc/db4733e531ecfbc0732f2aa10e2f6c01/TerritoryID_%E7%BB%93%E6%9E%9C.png)

    - 2-3. 非主索引鍵資料行上定義的非唯一叢集索引(non-unique clustered index which is defined on a non-primary key column)

        >`[AdventureWorks2014].[Sales].[Customer]` 已存在 `唯一叢集索引(unique clustered index)`，為了不影響原本 table 設計，就從 `[AdventureWorks2014].[Sales].[Customer]` 複製一份資料，接著建立`非唯一叢集索引(non-unique clustered index)` 來模擬

        ```sql
        USE [AdventureWorks2014]
        GO
        SELECT *
        INTO [AdventureWorks2014].[Sales].[Customer_test]
        FROM [AdventureWorks2014].[Sales].[Customer]
        Go
        CREATE CLUSTERED INDEX [ClusteredIndex_CustomerID] ON [Sales].[Customer_test]
        (
            [CustomerID] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)
        GO
        ```

        ```sql
        SELECT  
        [CustomerID]
            ,[PersonID]
            ,[StoreID]
            ,[TerritoryID]
            ,[AccountNumber]
            ,[rowguid]
            ,[ModifiedDate]
        FROM [AdventureWorks2014].[Sales].[Customer]
        where [CustomerID]=1
        ```

        ![non_unique_cluster](https://trello-attachments.s3.amazonaws.com/580432e4655468d3c9285abc/1200x92/6ec064d8c224942ff8b9d11e7cfc3c63/non_unique_cluster_%E7%BB%93%E6%9E%9C.png)

3. 當 N 是使用索引鍵找出索引鍵值之後，朝向分葉層級左側或右側啟動的不同搜尋/掃瞄次數時，掃瞄計數就是 N。  

    ```sql
    SELECT
    [CustomerID]
        ,[PersonID]
        ,[StoreID]
        ,[TerritoryID]
        ,[AccountNumber]
        ,[rowguid]
        ,[ModifiedDate]
    FROM [AdventureWorks2014].[Sales].[Customer]
    where [CustomerID] in (1,2,3,4,5)
    ```

    ![find5](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580432e4655468d3c9285abc/75e6dea9096f780bed174c0db4233db5/find5_%E7%BB%93%E6%9E%9C.png)

## logical reads(邏輯讀取)

1. 從資料快取中讀取的`page`(非筆數)。--> 會包含  `physical reads` 及 `read-ahead reads`
2. 數字愈小愈好  
3. 數字保持一致，是效能調校的重要參考

## physical reads(實體讀取)

1. 執行計畫生成後， cache 中缺少的資料從磁碟中讀取的頁數(page)。
2. 數字大多會變動，第二次之後的查詢，因資料已 cache 而變成 0
3. 對效能調校參考價值不大

## read-ahead reads(讀取前讀取)

1. 放入查詢快取中的頁數。
2. sql server 生成執行計畫過程中判斷查詢可能需要的資料而預先載入至 cache 的頁數(page)
3. 對效能調校參考價值不大

## lob logical reads(LOB 邏輯讀取)

1. 從資料快取中讀取的 text、ntext、image 或大數值類型 (varchar(max)、nvarchar(max)、varbinary(max)) 頁數。
2. 大型物件(large object)的邏輯讀取

## lob physical reads(LOB 實體讀取)

1. 從磁碟中讀取的 text、ntext、image 或大數值類型頁數。
2. 大型物件(large object)的實體讀取

## lob read-ahead reads(LOB 讀取前讀取)

1. 放入查詢快取中的 text、ntext、image 或大數值類型頁數。
2. 大型物件(large object)的讀取前讀取

## 參考資料

1. [SET STATISTICS IO(MSDN)](https://msdn.microsoft.com/en-us/library/ms184361.aspx)
2. [初談SQL Server邏輯讀、物理讀、預讀](http://www.2cto.com/database/201605/513734.html)
