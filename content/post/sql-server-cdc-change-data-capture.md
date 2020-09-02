---
title: "使用 SQL Server CDC (Change Data Capture) 來追蹤資料變更"
date: 2017-06-29T23:28:00+08:00
lastmod: 2018-09-23T23:28:12+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-cdc-change-data-capture"
aliases:
    - /2017/06/sql-server-cdc-change-data-capture.html
---
# 使用 SQL Server CDC (Change Data Capture) 來追蹤資料變更
同事在追查一個 issue，但追一追總是卡在 db 資料面，不知道 db 的資料變化歷程，所以遲遲找不出問題發生原因，所以想要在 table 上加 trigger 來紀錄資料被異動的歷程

查資料過程中，發現一個以前沒留意過的功能 CDC (Change Data Capture)，立馬來嘗試看看合不合用，隨手筆記一下

## 限制條件

只有下列版本 SQL Server 可以使用 CDC - Change Data Capture 功能

1.  SQL Server 2008 以上
2.  Enterprise
3.  Developer
4.  Enterprise Evaluation


![0notavailable](https://user-images.githubusercontent.com/3851540/27695525-bf959dd2-5d21-11e7-971f-530eacc226d1.png)

## 如何使用 SQL Server 範本

1.  SQL Server Management Studio (SSMS) 主選單 View --> Template Explorer

    ![1templateexplorer](https://user-images.githubusercontent.com/3851540/27695526-bf99a5ee-5d21-11e7-8f15-fc8ed19d3d77.png)

2.  Template Browser --> Change Data Capture

    ![2CDC](https://user-images.githubusercontent.com/3851540/27695527-bfb95f92-5d21-11e7-9ac2-fdeaadb5ea5c.png)

## 為 database 啟用 CDC

1.  Template

    ```sql
    -- ================================
    -- Enable Database for CDC Template
    -- ================================
    USE <Database_Name,sysname,Database_Name>
    GO
        EXEC sys.sp_cdc_enable_db
    GO
    ```

2.  範例

    ```sql
    -- ================================
    -- Enable Database for CDC Template
    -- ================================
    USE TestCDC
    GO
        EXEC sys.sp_cdc_enable_db
    GO
    ```

3.  成功啟用後在 System Table 下加入 CDC 相關的系統 table

    ![3CDCtable](https://user-images.githubusercontent.com/3851540/27695528-bfbd42a6-5d21-11e7-8da7-228ccd5b3a4a.png)

    *   cdc.captured_columns

        > 記錄 CDC 包含的欄位清單

    *   cdc.change_tables

        > 紀錄啟用 CDC 的 table 清單

    *   cdc.ddl_history

        > 紀錄啟用 CDC 後所有 DDL 操作歷程

    *   cdc.index_columns

        > 紀錄啟用 CDC table 的 index 資訊

    *   cdc.lsn_time_mapping

        > 紀錄 LSN 號碼跟時間

## 確認 database 是否啟用 CDC

1.  Template

    ```sql
    -- ========================================================
    -- Determine Whether a Database is Enabled for CDC Template
    -- ========================================================
    USE <Database_Name,sysname,Database_Name>
    GO
        SELECT is_cdc_enabled FROM sys.databases
        WHERE name = N'<Database_Name,sysname,Database_Name>'
    GO
    ```

2.  範例

    ```sql
    -- ========================================================
    -- Determine Whether a Database is Enabled for CDC Template
    -- ========================================================
    USE TestCDC
    GO
        SELECT is_cdc_enabled FROM sys.databases
        WHERE name = N'TestCDC'
    GO
    ```

## 為 table 啟用 CDC

1.  template

    ```sql
    -- ===============================================
    -- Enable a Table for All Changes Queries Template
    -- ===============================================
    USE <Database_Name,sysname,Database_Name>
    GO
        EXEC sys.sp_cdc_enable_table
        @source_schema = N'<source_schema,sysname,source_schema>',
        @source_name   = N'<source_name,sysname,source_name>',
        @role_name     = N'<role_name,sysname,role_name>',
        @supports_net_changes = 0
    GO
    ```

2.  範例

    ```sql
    -- ===============================================
    -- Enable a Table for All Changes Queries Template
    -- ===============================================
    USE TestCDC
    GO
        EXEC sys.sp_cdc_enable_table
        @source_schema = N'dbo',
        @source_name   = N'Users',
        @role_name     = N'CDCRole',
        @supports_net_changes = 0
    GO
    ```

3.  成功啟用後在 System Table 下加入 table 對應的 CDC table

    ![4CDCtable](https://user-images.githubusercontent.com/3851540/27695529-bfbed436-5d21-11e7-918c-7c581429c237.png)

## 確認 table 是否啟用 CDC

1.  template

    ```sql
    -- =====================================================
    -- Determine Whether a Table is Enabled for CDC Template
    -- =====================================================
    USE <Database_Name,sysname,Database_Name>
    GO
        SELECT is_tracked_by_cdc FROM sys.tables
        WHERE object_id = object_id(N'<source_schema,sysname,source_schema>.<source_name,sysname,source_name>')
    GO
    ```

2.  範例

    ```sql
    -- =====================================================
    -- Determine Whether a Table is Enabled for CDC Template
    -- =====================================================
    USE TestCDC
    GO
        SELECT is_tracked_by_cdc FROM sys.tables
        WHERE object_id = object_id(N'Users')
    GO
    ```

## 變更擷取

其中的 __$operation 數字意義如下
 
*   1 ==> Delete
*   2 ==> Insert
*   3 ==> Before Update
*   4 ==> After Update

![5capture](https://user-images.githubusercontent.com/3851540/27695530-bfda6796-5d21-11e7-9d1b-19400becaeb8.png)



## 停用 table CDC 功能

1.  template

    ```sql
    -- ===============================================
    -- Disable a Capture Instance for a Table Template
    -- ===============================================
    USE <Database_Name,sysname,Database_Name>
    GO
        EXEC sys.sp_cdc_disable_table
        @source_schema = N'<source_schema,sysname,source_schema>',
        @source_name   = N'<source_name,sysname,source_name>',
        @capture_instance = N'<capture_instance,sysname,capture_instance>'
    GO
    ```

2.  範例

    ```sql
    -- ===============================================
    -- Disable a Capture Instance for a Table Template
    -- ===============================================
    USE TestCDC
    GO
        EXEC sys.sp_cdc_disable_table
        @source_schema = N'dbo',
        @source_name   = N'Users',
        @capture_instance = N'dbo_Users'
    GO
    ```
3.  <span style="color:red">停用特定 table CDC 功能後，對應的 CDC table 就會被刪除，這個要特別留意</span>


## 停用 database CDC 功能

1.  template

    ```sql
    -- =================================
    -- Disable Database for CDC Template
    -- =================================
    USE <Database_Name,sysname,Database_Name>
    GO
        EXEC sys.sp_cdc_disable_db
    GO
    ```

2.  範例：

    ```sql
    -- =================================
    -- Disable Database for CDC Template
    -- =================================
    USE TestCDC
    GO
        EXEC sys.sp_cdc_disable_db
    GO
    ```

3.  與停用 table CDC 類似，停用 database CDC 功能也會刪除相關 CDC table


## 心得

透過幾個簡單設定，馬上就能達到資料備份的功用，SQL Server 真是友善呀

雖然簡單幾個動作就可以達到資料備份，但還沒有真正確認對效能的影響程度前還是不該隨意在 production server 上啟用，加上只有基本欄位還是不夠的，要再找找看有沒有客製空間，否則還是只能用舊方法 - Trigger 進行了，但至少多了個選擇可用

# 參考資訊

1.  [關於異動資料擷取 (SQL Server)](https://docs.microsoft.com/zh-tw/sql/relational-databases/track-changes/about-change-data-capture-sql-server?WT.mc_id=DOP-MVP-5002594)
2.  [Introduction to Change Data Capture (CDC) in SQL Server 2008](https://www.simple-talk.com/sql/learn-sql-server/introduction-to-change-data-capture-cdc-in-sql-server-2008/)
