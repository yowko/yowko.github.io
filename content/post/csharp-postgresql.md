---
title: "使用 C# 存取 PostgreSQL"
date: 2019-02-17T20:40:00+08:00
lastmod: 2021-11-03T20:40:30+08:00
draft: false
tags: ["csharp","NoSQL","PostgreSQL"]
slug: "csharp-postgresql"
---
## 使用 C# 存取 PostgreSQL

之前筆記 [使用 C# 存取 Cassandra](https://blog.yokwo.com/csharp-cassandra) 提到想要將 log 存放至 NoSQL 中而正在嘗試某幾套 NoSQL，現在就來看看 PostgreSQL 的使用吧

## 基本環境說明

> 在 mac 上使用 docker 建立 PostgreSQL，接著使用 C# 連線至 PostgreSQL 執行基本 CRUD

1. macOS Mojave 10.14.2
2. Docker Community 18.09.1
3. Bogus 25.0.4
4. PostgreSQL 11.2
5. Npgsql 4.0.4

## 建立 PostgreSQL instance

> 透過 docker 僅建立單一 local PostgreSQL db instance，並設定預設帳號 `postgres` 密碼為 `pass.123`

```bash
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=pass.123 postgres
```

## 使用方式

1. 建立 Database

    ```sql
    create database benchmark;
    ```

2. 建立 Schema

    ```sql
    create schema test;
    ```

3. 建立 Table

    > 欄位使用 jsonb 是讀取快，json 則是寫入快，不過 json 欄位在 update 時沒有 `jsonb_set` 這類函數可用，針對 json 的更新我還沒找到簡易做法

    ```sql
    create table test.Users
    (
        Uid serial not null,
        "User" Jsonb
    );

    create unique index Users_Uid_uindex
        on test.Users (Uid);

    alter table test.Users
        add constraint Users_pk
            primary key (Uid);
    ```

4. 安裝 `Npgsql` NuGet 套件 : [Npgsql - .NET Access to PostgreSQL](http://www.npgsql.org/)

    > 本次測試使用 Npgsql  版本為 `4.0.4`

    - Package Manager

        ```cmd
        Install-Package Npgsql
        ```

    - .NET CLI

        ```cmd
        dotnet add package Npgsql
        ```

5. 實際存取 PostgreSQL

    - Insert

        ```cs
        //準備連線字串
        var connString = "Host=localhost;Port=5432;Username=postgres;Password=pass.123;Database=benchmark";

        using (var conn = new NpgsqlConnection(connString))
        {
            conn.Open();
            //準備 insert 用假資料
            var _user = GetFakeUserData();
                
            //透過 binary 方式來寫入 json (其中 User 欄位因為是保留字需要跳脫)
            using (var writer =conn.BeginBinaryImport("COPY test.users (\"User\") FROM STDIN (FORMAT BINARY)"))
            {
                //開始 row
                writer.StartRow();
                //將欲新增的 row 內容逐一寫入
                writer.Write(JsonConvert.SerializeObject(_user), NpgsqlDbType.Jsonb);
                //row 結束
                writer.Complete();
            }
        }
        ```

    - Select

        ```cs
        //準備連線字串
        var connString = "Host=localhost;Port=5432;Username=postgres;Password=pass.123;Database=benchmark";

        using (var conn = new NpgsqlConnection(connString))
        {
            conn.Open();
            //依 jsonb 欄位過濾 Name 屬性資料
            using (var cmd = new NpgsqlCommand("SELECT * FROM test.users WHERE \"User\"->> 'Name' = @name; ", conn))
            {
                //直接條件內容
                cmd.Parameters.AddWithValue("name", "Yowko");
                using (var reader = cmd.ExecuteReader())
                    while (reader.Read())
                        //取得欄位 id 輸出
                        reader.GetInt32(0).Dump();
            }
        }
        ```

    - Update

        ```cs
        //準備連線字串
        var connString = "Host=localhost;Port=5432;Username=postgres;Password=pass.123;Database=benchmark";

        using (var conn = new NpgsqlConnection(connString))
        {
            conn.Open();
            using (var cmd = new NpgsqlCommand())
            {
                cmd.Connection = conn;
                //準備更新語法，將 jsonb 中的 Name 更新為 "Yowko"
                cmd.CommandText = "UPDATE test.users SET \"User\" = jsonb_set(\"User\",'{Name}','\"Yowko\"'::jsonb,false) WHERE \"User\"->> 'Name' = @name; ";
                //加入過濾條件
                cmd.Parameters.AddWithValue("name", "old name");
                cmd.ExecuteNonQuery();
            }
        }
        ```

    - Delete

        ```cs
        //準備連線字串
        var connString = "Host=localhost;Port=5432;Username=postgres;Password=pass.123;Database=benchmark";

        using (var conn = new NpgsqlConnection(connString))
        {
            conn.Open();
            using (var cmd = new NpgsqlCommand())
            {
                cmd.Connection = conn;
                //準備刪除語法
                using (var cmd = new NpgsqlCommand("DELETE FROM test.users WHERE \"User\"->> 'Name' = @name; ", conn))
                {
                    //加入過濾條件
                    cmd.Parameters.AddWithValue("name", "Yowko");
                    cmd.ExecuteNonQuery();
                }
            }
        }
        ```

## 心得

PostgreSQL 在普遍印象中是套 RDBMS - 關聯式資料庫管理系統，但因為 open source 的特性，讓許多社群及貢獻者得以持續改善，所以在儲存 json 的功能上有著不輸其他 NoSQL 的表現

但就 C# client - Npgsql 的使用上來看，api 的操作上非常接近 ADO.NET：自行連線控制、手動準備 sql 指令及實際執行都非常相似，但近幾年多數 .NET 開發人員比較習慣的 LINQ 式操作在 Npgsql 上表現並不好

## 參考資訊

1. [PostgreSQL Driver Quick Tour](http://PostgreSQL.github.io/mongo-csharp-driver/2.7/getting_started/quick_tour/)
2. [PostgreSQL/mongo-csharp-driver](https://github.com/PostgreSQL/mongo-csharp-driver)
3. [Getting Started](https://www.npgsql.org/doc/index.html)
