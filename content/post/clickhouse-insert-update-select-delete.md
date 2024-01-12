---
title: "新增修改刪除查詢 ClickHouse 資料"
date: 2024-01-10T00:30:00+08:00
lastmod: 2024-01-10T00:30:31+08:00
draft: false
tags: ["docker","csharp","clickhouse"]
slug: "clickhouse-insert-update-select-delete"
---

## 新增修改刪除查詢 ClickHouse 資料"

之前筆記 [C# 如何新增資料至 ClickHouse](/csharp-insert-clickhouse) 與 [ClickHouse 在彙總資料時的效能優勢](/clickhouse-aggregate-query-performance/) 分別紀錄到如何透過 C# 來新增大量資料至 ClickHouse 以及如何查詢 ClickHouse 資料，整理筆記時總覺得應該把其他操作一併紀錄一下，所以今天來看看如何更新、刪除、修改 ClickHouse 資料並且加上使用 c# 的範例

## 基本環境說明

1. macOS Sonoma 14.2.1 (Apple M2 Pro)
2. OrbStack Version 1.2.0 (16496)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.2
5. Container Images

    - clickhouse/clickhouse-server:23.11.3.23

6. NuGet Library

    - Bogus 35.3.0
    - ClickHouse.Client 6.8.1

7. docker-compose.yml

    {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "docker-compose.yaml">}}

8. ClickHouse DDL

    {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "ddl.sql">}}

## 基本操作

以下主要是使用 c# 來執行，但如果需要直接執行 sql script 也可以透過 clickhouse 內建的 web ui 來執行：`http://localhost:8123/play`

1. 新增

    - 1-1. 單筆新增

        - sql script

            ```sql
            insert into test.orders (
                id ,
                order_date,
                product_id,
                order_type,
                amount
            ) Values (1234567890,'2023/01/01',100,1,100.001)
            ```

        - 使用 c#

            {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "insert-single.cs">}}

    - 1-2. 新增多筆

        詳細內容可以參考之前筆記 [C# 如何新增資料至 ClickHouse](/csharp-insert-clickhouse)

        {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "insert-bulk.cs">}}

2. 查詢

    - sql script

        ```sql
        select * from test.orders;
        ```

    - C# 程式碼

        {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "query.cs">}}

3. 更新

    - 語法

        - sql script

            ```sql
            ALTER TABLE [<database>.]<table> UPDATE <column> = <expression> WHERE <filter_expr>
            ```

        - 實際範例

            ```sql
            ALTER TABLE test.orders UPDATE amount = 100.001 WHERE id= 1882682734613504000;
            ```

    - C# 程式碼

        {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "update.cs">}}

4. 刪除

    - 3-1. 刪除條件符合的資料

        - 語法
            - sql script

                ```sql
                ALTER TABLE [<database>.]<table> DELETE WHERE <filter_expr>
                ```

            - 實際範例

                ```sql
                ALTER TABLE test.orders DELETE WHERE d= 1882682734613504000
                ```

        - C# 程式碼

            {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "delete-single.cs">}}

    - 3-2. 快速刪除 table 中所有資料

        - 語法
            - sql script

                ```sql
                TRUNCATE TABLE [<database>.]<table>
                ```

            - 實際範例

                ```sql
                TRUNCATE TABLE test.orders
                ```

        - C# 程式碼

            {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "deletetruncate.cs">}}

    - 3-3. 輕量刪除資料

        - 語法
            - sql script

                ```sql
                DELETE FROM [db.]table [ON CLUSTER cluster] [WHERE expr]
                ```

            - 實際範例

                ```sql
                DELETE FROM test.orders WHERE id= 1882682734613504000
                ```

        - C# 程式碼

            {{<gist yowko a31678a9041841577ebe3d4b7a0ad023 "deletelightweight.cs">}}

## 心得

我在測試語法以及 c# 程式碼時，發現 ClickHouse primary key 跟過去使用的資料庫有些不同：

1. create table 必需要有的是 order by，但是不一定要有 primary key
2. 如果沒有明確指定 primary key，預設會使用 order by 的欄位當作 primary key
3. primary key 主要負責 index 的建立，因此允許重複，並不會出現 pk 衝突的錯誤

原始程式碼請參考：[GitHub:yowko/ClickHouseCRUD](https://github.com/yowko/ClickHouseCRUD)

## 參考資訊

1. [Inserting Data into ClickHouse](https://clickhouse.com/docs/en/guides/inserting-data)
2. [SELECT Queries in ClickHouse](https://clickhouse.com/docs/en/guides/writing-queries)
3. [Updating and Deleting ClickHouse Data](https://clickhouse.com/docs/en/guides/developer/mutations)
4. [C# 如何新增資料至 ClickHouse](/csharp-insert-clickhouse)
5. [ClickHouse 在彙總資料時的效能優勢](/clickhouse-aggregate-query-performance/)
6. [ClickHouse 建表create table时primary by与order by](https://blog.csdn.net/qq_36951116/article/details/106260189)
7. [GitHub:yowko/ClickHouseCRUD](https://github.com/yowko/ClickHouseCRUD)
