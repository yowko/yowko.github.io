---
title: "C# 如何快速新增大量資料至 MySQL"
date: 2023-12-22T00:30:00+08:00
lastmod: 2023-12-22T00:30:31+08:00
draft: false
tags: ["mysql","csharp"]
slug: "csharp-mysql-bulk-insert"
---

## C# 如何快速新增大量資料至 MySQL

最近在重現系統上遇到的狀況，初步懷疑是資料量過大，造成相關處理效能不佳，而連帶影響到系統後續運作，但這是初步懷疑，為了印證這個懷疑，首先需要產生大量資料，透過 C# 搭配 Bogus 產生假資料後再 insert 進 MySql 實在太耗費時間了，所以今天就來紀錄利用 MySqlBulkLoader 來快速 bulk insert 大量資料的方式。

MySqlBulkLoader 是 MySQL Connector/NET 提供的一個類別，可以快速將資料 insert 進 MySql MySqlBulkLoader 可以透過 csv 與 stream 將資料寫入 MySql

## 基本環境說明

1. macOS Sonoma 14.2.1 (Apple M2 Pro)
2. OrbStack Version 1.2.0 (16496)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.2
5. Container Images

    - mysql:8.2

6. NuGet Library

    - Bogus 35.0.1
    - MySqlConnector 2.3.1
    - BenchmarkDotNet 0.13.11
    - CsvHelper 30.0.1

7. docker-compose.yml

    {{<gist yowko 4e41fa5ac3a825d3293a5c0910b8112d "docker-compose.yml">}}

8. mysql DDL

    {{<gist yowko 4e41fa5ac3a825d3293a5c0910b8112d "ddl.sql">}}

## 設定方式

1. C# 連線字串設定：需加上 `AllowLoadLocalInfile=true;`

    - 語法：

        ```txt
        Server={ip or host};Port={port};Database={db name};Username={username};Password={password};Allow User Variables=true;AllowLoadLocalInfile=true;
        ```

    - 範例：

        ```txt
        Server=localhost;Port=3306;Database=test;Username=root;Password=pass.123;Allow User Variables=true;AllowLoadLocalInfile=true;"
        ```

    - 未設定錯誤

        - 錯誤訊息

            ```log
            Unhandled exception. System.NotSupportedException: To use MySqlBulkLoader.Local=true, set AllowLoadLocalInfile=true in the connection string. See https://fl.vu/mysql-load-data
               at MySqlConnector.MySqlBulkLoader.LoadAsync(IOBehavior ioBehavior, CancellationToken cancellationToken) in /_/src/MySqlConnector/MySqlBulkLoader.cs:line 205
               at MySqlConnector.MySqlBulkLoader.Load() in /_/src/MySqlConnector/MySqlBulkLoader.cs:line 146
               at Program.<<Main>$>g__StoreToMysql|0_1(Order[] orders) in /Users/yowko.tsai/POCs/ClickHouse/DataGenerator/Program.cs:line 190
               at Program.<<Main>$>g__GenTestRecords|0_0(Int32 batchcount, Int32 batchsize) in /Users/yowko.tsai/POCs/ClickHouse/DataGenerator/Program.cs:line 127
               at Program.<Main>$(String[] args) in /Users/yowko.tsai/POCs/ClickHouse/DataGenerator/Program.cs:line 52
               at Program.<Main>(String[] args)
            ```

        - 錯誤截圖

            ![1connectionstring](https://github.com/yowko/picsbed/assets/3851540/320b917c-6345-4f08-92c6-661cb98ebe5b)

2. MySql 設定啟用 `local_infile`

    - 語法：

        ```sql
        SET GLOBAL local_infile = 1;
        ```

    - 未設定錯誤

        - 錯誤訊息

            ```log
            Unhandled exception. MySqlConnector.MySqlException (0x80004005): Loading local data is disabled; this must be enabled on both the client and server sides
               at MySqlConnector.Core.ServerSession.ReceiveReplyAsync(IOBehavior ioBehavior, CancellationToken cancellationToken) in /_/src/MySqlConnector/Core/ServerSession.cs:line 936
               at MySqlConnector.Core.ResultSet.ReadResultSetHeaderAsync(IOBehavior ioBehavior) in /_/src/MySqlConnector/Core/ResultSet.cs:line 37
               at MySqlConnector.MySqlDataReader.ActivateResultSet(CancellationToken cancellationToken) in /_/src/MySqlConnector/MySqlDataReader.cs:line 130
               at MySqlConnector.MySqlDataReader.InitAsync(CommandListPosition commandListPosition, ICommandPayloadCreator payloadCreator, IDictionary`2 cachedProcedures, IMySqlCommand command, CommandBehavior behavior, Activity activity, IOBehavior ioBehavior, CancellationToken cancellationToken) in /_/src/MySqlConnector/MySqlDataReader.cs:line 483
               at MySqlConnector.Core.CommandExecutor.ExecuteReaderAsync(CommandListPosition commandListPosition, ICommandPayloadCreator payloadCreator, CommandBehavior behavior, Activity activity, IOBehavior ioBehavior, CancellationToken cancellationToken) in /_/src/MySqlConnector/Core/CommandExecutor.cs:line 56
               at MySqlConnector.MySqlCommand.ExecuteNonQueryAsync(IOBehavior ioBehavior, CancellationToken cancellationToken) in /_/src/MySqlConnector/MySqlCommand.cs:line 309
               at MySqlConnector.MySqlBulkLoader.LoadAsync(IOBehavior ioBehavior, CancellationToken cancellationToken) in /_/src/MySqlConnector/MySqlBulkLoader.cs:line 212
               at MySqlConnector.MySqlBulkLoader.Load() in /_/src/MySqlConnector/MySqlBulkLoader.cs:line 146
               at Program.<<Main>$>g__StoreToMysql|0_1(Order[] orders) in /Users/yowko.tsai/POCs/ClickHouse/DataGenerator/Program.cs:line 190
               at Program.<<Main>$>g__GenTestRecords|0_0(Int32 batchcount, Int32 batchsize) in /Users/yowko.tsai/POCs/ClickHouse/DataGenerator/Program.cs:line 127
               at Program.<Main>$(String[] args) in /Users/yowko.tsai/POCs/ClickHouse/DataGenerator/Program.cs:line 52
               at Program.<Main>(String[] args)
            ```

        - 錯誤截圖

            ![2localinfile](https://github.com/yowko/picsbed/assets/3851540/f9a50142-302e-409c-a0b0-2c19a19f5d93)

3. C# 程式碼

    - 使用 csv

        > 使用 csv 需要將資料寫入暫存檔案，再將暫存檔案路徑傳入 `MySqlBulkLoader`，這邊使用 `CsvHelper` 來產生 csv 檔案

        {{<gist yowko 4e41fa5ac3a825d3293a5c0910b8112d "MySqlBulkLoaderCsv.cs" >}}

    - 使用 stream

        > 使用 stream 需要將資料寫入 `MemoryStream`，再將 `MemoryStream` 傳入 `MySqlBulkLoader`，可能 memory 佔用量會比較大，但不需要產生暫存檔案

        {{<gist yowko 4e41fa5ac3a825d3293a5c0910b8112d "MySqlBulkLoaderStream.cs" >}}

4. 完整程式碼

    {{<gist yowko 4e41fa5ac3a825d3293a5c0910b8112d "Program.cs" >}}

## 心得

1. client 的連線字串與 MySql server 都要額外設定才能使用 `MySqlBulkLoader`
2. 相較於 MySql .NET library 沒有 bulk insert 速度提升非常多 (以我的測試來看從 1000 筆資料的 80 倍差距，到 100000 筆資料的 205 倍差距)
3. csv 與 stream 兩種方式都可以使用，但 stream 在資料量較多時會快一些也不需要額外處理暫存檔，但會佔用較多記憶體

- 執行速度與 memory 用量比較

    ![3benchmark](https://github.com/yowko/picsbed/assets/3851540/9afcc078-9047-4696-82b8-8946789fb3b8)

完整程式碼：[GitHub:yowko/MySqlBulkInsert](https://github.com/yowko/MySqlBulkInsert)

## 參考資訊

1. [C# - How to bulk insert into MySQL using C#](https://itecnote.com/tecnote/c-how-to-bulk-insert-into-mysql-using-c/)
2. [Using the MySqlBulkLoader Class](https://dev.mysql.com/doc/connector-net/en/connector-net-programming-bulk-loader.html)
3. [Using Load Data Local Infile](https://mysqlconnector.net/troubleshooting/load-data-local-infile/)
4. [C# - CsvHelper not writing anything to memory stream](https://itecnote.com/tecnote/c-csvhelper-not-writing-anything-to-memory-stream/)
5. [MySqlBulkLoader一种高效的数据保存方案](https://blog.csdn.net/sD7O95O/article/details/130355003)
6. [GitHub:yowko/MySqlBulkInsert](https://github.com/yowko/MySqlBulkInsert)
