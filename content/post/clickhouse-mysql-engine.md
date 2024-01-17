---
title: "ClickHouse 使用 MySql Database Engine"
date: 2024-01-15T00:30:00+08:00
lastmod: 2024-01-15T00:30:31+08:00
draft: false
tags: ["docker","csharp","clickhouse","mysql"]
slug: "clickhouse-mysql-engine"
---

## ClickHouse 使用 MySql Database Engine

之前筆記 [ClickHouse 在彙總資料時的效能優勢](/clickhouse-aggregate-query-performance) 紀錄到 ClickHouse 與 MySql 在特定查詢情境的效能差異，查閱官網文件時，發現 ClickHouse 除了支援自有的 table engine 之外，也支援其他 DB 的 engine 其中還有兩種類型：`Database Engine` 與 `Table Engine`，所以今天就來紀錄一下 ClickHouse 如何使用 MySql Database Engine 與 MySql Table Engine 之間的效能差異

## 基本環境說明

1. macOS Sonoma 14.2.1 (Apple M2 Pro)
2. OrbStack Version 1.2.0 (16496)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.2
5. Container Images

    - clickhouse/clickhouse-server:23.11.3.23
    - mysql:8.2

6. NuGet Library

    - BenchmarkDotNet 0.13.12
    - ClickHouse.Client 7.0.0
    - MySql.Data 8.3.0

7. docker-compose.yml

    {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "docker-compose.yml">}}

8. MySql DDL

    {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "mysql-ddl.sql">}}

9. 測試資料

    > 產生一億筆資料

    {{<gist yowko 3920ce5f4147999323361744dfd04247 "DataGenProgram.cs">}}

## 設定方式

1. Database Engine

    詳細內容請參考官網文件 [DatabaseEngines:MySQL](https://clickhouse.com/docs/en/engines/database-engines/mysql)

    - 語法

        {{<gist yowko 3920ce5f4147999323361744dfd04247 "database-engine.sql">}}

    - 範例

        {{<gist yowko 3920ce5f4147999323361744dfd04247 "database-engine-example.sql">}}

2. Table Engine

    詳細內容請參考官網文件 [TableEngin:MySQL](https://clickhouse.com/docs/en/engines/table-engines/integrations/mysql)

    - 語法

        {{<gist yowko 3920ce5f4147999323361744dfd04247 "table-engine.sql">}}

    - 範例

        {{<gist yowko 3920ce5f4147999323361744dfd04247 "table-engine-example.sql">}}

## 效能比較

1. 程式碼

    {{<gist yowko 3920ce5f4147999323361744dfd04247 "Program.cs">}}

2. 比較結果

    ![1benchmark](https://github.com/yowko/picsbed/assets/3851540/4862d103-5764-40d6-bee4-f53a243d4299)

## 心得

1. 僅支援以下 data type，其他 MySQL data type 皆會被視為 string，使用時需要自行做型別轉換，難免會有效能耗損的問題

    MySQL|ClickHouse
    :---|:---
    UNSIGNED TINYINT|UInt8
    TINYINT|Int8
    UNSIGNED SMALLINT|UInt16
    SMALLINT|Int16
    UNSIGNED INT, UNSIGNED MEDIUMINT|UInt32
    INT, MEDIUMINT|Int32
    UNSIGNED BIGINT|UInt64
    BIGINT|Int64
    FLOAT|Float32
    DOUBLE|Float64
    DATE|Date
    DATETIME, TIMESTAMP|DateTime
    BINARY|FixedString

2. 原本想要比照之前筆記 ：依時間區間過濾資料，但我嘗試了 `BETWEEN` 與 `>=` 跟 `<=`，都無法正確查到資料，不知道是我使用方式不對，還是 mysql engine 對於 date 的比對有額外的處理方式，所以就先不將這個部分納入比較

3. 雖然簡單的 `WHERE` 相關條件 (`=`, `!=`, `>`, `>=`, `<`, `<=`) 是在 MySQL 上執行，但相同查詢在 ClickHouse 相較於 MySQL 快上近 50%，雖然比不上直接查詢 ClickHouse 動輒幾十甚至上百倍的差距，但也不失為一個選擇

4. ClickHouse MySql engine (database engine 與 table engine) 都支援 `SELECT` 與 `INSERT` 指令，但 `RENAME`、`CREATE TABLE`、`ALTER` 則不行

## 參考資訊

1. [ClickHouse 在彙總資料時的效能優勢](/clickhouse-aggregate-query-performance)
2. [DatabaseEngines:MySQL](https://clickhouse.com/docs/en/engines/database-engines/mysql)
3. [TableEngin:MySQL](https://clickhouse.com/docs/en/engines/table-engines/integrations/mysql)
