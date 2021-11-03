---
title: "關於清除 SQL Server 查詢快取的那些事"
date: 2016-12-27T00:42:34+08:00
lastmod: 2021-10-26T00:42:34+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-clear-cache"
aliases:
    - /2016/12/sql-server-clear-buffer.html
---
## 關於清除 SQL Server 查詢快取的那些事

之前的筆記 [試著學會看懂 SQL Server IO 統計資訊](/sql-server-io) 中，有粗略地介紹 SQL Server IO 的統計資訊，其中 `logical reads(邏輯讀取)` 是我們用來進行效能調校的重要依據，文中也提到 cache 來源是 `physical reads(實體讀取)`、`read-ahead reads(讀取前讀取)` 。在為了比較使用 memory 與使用 disk 的查詢時間差異，就得透過清除查詢 cache 來進行，接著就來看看該如何清除查詢 cache.

## 差異

1. 第一次查詢(使用 disk)

    ![fromdisk](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x293/fa7925d681f1e919127af69b7d3960c8/_output_fromdisk.png)

2. 第二次以後查詢(使用 memory)

    ![frommemory](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x293/91374b15f91e5d4bcf5d4b0dc460b382/_output_frommemory.png)

## 名詞介紹

1. Clean Buffer

    > data page cache 沒有修改過的

2. Dirty Buffer

    > data page cache 已修改過但未被寫入至磁碟中的部份

3. Cold Buffer Cache

    > data page 還沒載入 memory 中，需要從磁碟讀取

## 清除 Cache 語法

```sql
CHECKPOINT;
DBCC DROPCLEANBUFFERS;
```

## 關於 `DBCC DROPCLEANBUFFERS`

- 僅清除 `Clean Buffer`，`Dirty Buffer`無法被清除
- 語法
  - Syntax for SQL Server

    ```sql
    DBCC DROPCLEANBUFFERS [ WITH NO_INFOMSGS ]
    ```

  - Syntax for Azure SQL Data Warehouse and Parallel Data Warehouse

    ```sql
    DBCC DROPCLEANBUFFERS ( COMPUTE | ALL ) [ WITH NO_INFOMSGS ]
    ```

- 參數說明
  
  - WITH NO_INFOMSGS

    >隱藏所有參考訊息。`Azure SQL Data Warehouse` 跟 `Parallel Data Warehouse` 環境下預設隱藏

  - COMPUTE
    - 僅 `Azure SQL Data Warehouse` 跟 `Parallel Data Warehouse` 有效
    - 清除計算節點 cache

  - ALL
    - 僅 `Azure SQL Data Warehouse` 跟 `Parallel Data Warehouse` 有效
    - 清除所有節點 cache
    - 預設值

## 為什麼需要 CHECKPOINT

> `CHECKPOINT` 會將 `Dirty Buffer` 強制寫入 disk

我們來模擬`Dirty Buffer` 的情境如下

1. 一般的 select
    - 將資料載入 cache

        ```sql
        SELECT * FROM [AdventureWorks2014].[Sales].[Customer]
        ```

2. 檢查 cache
    - cache 有 122 筆資料

        ```sql
        select sysObj.name,* 
        from sys.dm_os_buffer_descriptors bufferDescriptors
        INNER JOIN sys.allocation_units AllocUnits ON bufferDescriptors.allocation_unit_id = AllocUnits.allocation_unit_id
        INNER JOIN sys.partitions Partitions ON AllocUnits.container_id = Partitions.hobt_id
        INNER JOIN sys.objects sysObj ON Partitions.object_id = sysObj.object_id
        WHERE bufferDescriptors.database_id = DB_ID()
        AND sysObj.is_ms_shipped = 0
        ```

        ![NORAMLCACHE](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x422/d8ef56c8a7974dbcc936847ef5ff6f2c/_output_NORAMLCACHE.png)

3. 使用 `DBCC DROPCLEANBUFFERS`

    ```sql
    DBCC DROPCLEANBUFFERS
    ```

4. 檢查 cache
    - 因為僅有 `Clean Buffer`，所以可以全部清除

        ```sql
        select sysObj.name,*
        from sys.dm_os_buffer_descriptors bufferDescriptors
        INNER JOIN sys.allocation_units AllocUnits ON bufferDescriptors.allocation_unit_id = AllocUnits.allocation_unit_id
        INNER JOIN sys.partitions Partitions ON AllocUnits.container_id = Partitions.hobt_id
        INNER JOIN sys.objects sysObj ON Partitions.object_id = sysObj.object_id
        WHERE bufferDescriptors.database_id = DB_ID()
        AND sysObj.is_ms_shipped = 0
        ```

        ![cleanbufferdbcc](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x423/92ee3f137d2561a94494aa7d6509dd69/_output_cleanbufferdbcc.png)

5. 修改一筆資料
    - 製造出 `Dirty buffer`

        ```sql
        UPDATE [Sales].[Customer]
        SET [ModifiedDate] = GETDATE()
        WHERE [CustomerID]=1
        GO
        ```

6. 一般的 select
    - 將資料載入 cache

        ```sql
        SELECT * FROM [AdventureWorks2014].[Sales].[Customer]
        ```

7. 檢查 cache
    - cache 數量不變， 有 122 筆資料

        ```sql
        select sysObj.name,* 
        from sys.dm_os_buffer_descriptors bufferDescriptors
        INNER JOIN sys.allocation_units AllocUnits ON bufferDescriptors.allocation_unit_id = AllocUnits.allocation_unit_id
        INNER JOIN sys.partitions Partitions ON AllocUnits.container_id = Partitions.hobt_id
        INNER JOIN sys.objects sysObj ON Partitions.object_id = sysObj.object_id
        WHERE bufferDescriptors.database_id = DB_ID()
        AND sysObj.is_ms_shipped = 0
        ```

        ![update](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x399/cfd2ebab5ff062f07525f2c160c77944/_output_update.png)

8. 使用 `DBCC DROPCLEANBUFFERS`

    ```sql
    DBCC DROPCLEANBUFFERS
    ```

9. 檢查 cache
    - 因為有 `Dirty Buffer`，cache 無法全部清除

        ```sql
        select sysObj.name,* 
        from sys.dm_os_buffer_descriptors bufferDescriptors
        INNER JOIN sys.allocation_units AllocUnits ON bufferDescriptors.allocation_unit_id = AllocUnits.allocation_unit_id
        INNER JOIN sys.partitions Partitions ON AllocUnits.container_id = Partitions.hobt_id
        INNER JOIN sys.objects sysObj ON Partitions.object_id = sysObj.object_id
        WHERE bufferDescriptors.database_id = DB_ID()
        AND sysObj.is_ms_shipped = 0
        ```

        ![updatedbcc](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x408/679853a1d4d7301c5cba58b39fc7cd57/_output_updatedbcc.png)

10. 使用 `CHECKPOINT` 及 `DBCC DROPCLEANBUFFERS`

    ```sql
    CHECKPOINT;
    DBCC DROPCLEANBUFFERS ;
    ```

11. 檢查 cache
    - `CHECKPOINT` 將 `Dirty buffer` 被寫入 disk
    - `DBCC DROPCLEANBUFFERS`清除 `Clean buufer`
    - cache 即可全數清除

        ```sql
        select sysObj.name,* 
        from sys.dm_os_buffer_descriptors bufferDescriptors
        INNER JOIN sys.allocation_units AllocUnits ON bufferDescriptors.allocation_unit_id = AllocUnits.allocation_unit_id
        INNER JOIN sys.partitions Partitions ON AllocUnits.container_id = Partitions.hobt_id
        INNER JOIN sys.objects sysObj ON Partitions.object_id = sysObj.object_id
        WHERE bufferDescriptors.database_id = DB_ID()
        AND sysObj.is_ms_shipped = 0
        ```

        ![ckdbcc](https://trello-attachments.s3.amazonaws.com/580af37595015aea5f142a0b/1200x408/4b3b68cb0702af0a11b14cb310d93140/_output_ckdbcc.png)

## 參考資料

1. [SQL Server: What is a COLD, DIRTY or CLEAN Buffer?](https://blogs.msdn.microsoft.com/psssql/2009/03/17/sql-server-what-is-a-cold-dirty-or-clean-buffer/)
2. [DBCC DROPCLEANBUFFERS and CHECKPOINT](https://sqldbpool.com/2012/08/18/dbcc-dropcleanbuffers-and-checkpoint/)
3. [DB_ID](https://msdn.microsoft.com/zh-tw/library/ms186274.aspx)
4. [Checkpoint](https://msdn.microsoft.com/zh-tw/library/ms189573.aspx)
5. [DBCC DROPCLEANBUFFERS](https://msdn.microsoft.com/zh-tw/library/ms187762.aspx)
