---
title: "使用 C# 存取 Cassandra"
date: 2019-02-09T23:45:00+08:00
lastmod: 2019-02-09T23:44:30+08:00
draft: false
tags: ["C#","NoSQL"]
slug: "csharp-cassandra"
---
# 使用 C# 存取 Cassandra
公司專案因為流量龐大連帶也會產生大量 log，過去都是使用 local file 來儲存，但在 cluster 的環境下 log file 會散落在許多主機上，一旦需要查閱詳細內容或是追查 issue 就是苦工也很浪費時間，因此希望可以更有效率地存取 log、讓 log 也可以更有系統化的具備 HA 機制所以考慮將 log 儲存至 NoSQL 中，至於 NoSQL 百家爭鳴，該用哪一套則還沒有定論，就看實際效能表現再決定，首先就先來看看 Cassandra

## 基本環境說明

> 在 mac 上使用 docker 建立 Cassandra，接著使用 C# 連線至 Cassandra 執行基本 CRUD

1. macOS Mojave 10.14.2
2. Docker Community 18.09.1
3. Bogus 25.0.4
4. Cassandra 3.11.3
5. CassandraCSharpDriver 3.7.0

## 建立 Cassandra instance

> 透過 docker 僅建立單一 local Cassandra db instance

```bash
docker run -d -p 9042:9042 cassandra
```

## 使用方式
1. 建立 keyspace (如同一般資料庫的 database)

    > 如 keyspace 不存在就建立，僅建立於單一節點的 cluster 中

    ```
    CREATE KEYSPACE benchmark
    WITH REPLICATION = { 
    'class' : 'SimpleStrategy', 
    'replication_factor' : 1 
    };
    ```

2. 建立 table 

    >需預先定義資料表結構

    ```
    CREATE TABLE IF NOT EXISTS benchmark.user
    (
        userid uuid primary key,
        birthday timestamp,
        currentsalary decimal,
        name text
    )
    ```
3. 安裝 NuGet 套件：
    - Package Manager

        ```
        Install-Package CassandraCSharpDriver
        ``` 
    - .NET CLI

        ```
        dotnet add package CassandraCSharpDriver
        ```
4. 實際存取 Cassandra

    - Insert

        ```cs
        //準備 insert 用假資料
        var _user = GetFakeUserData();

        //準備 cassandra 的連線資訊
        var cluster = Cluster.Builder()
        .AddContactPoints("127.0.0.1")
        .WithPort(9042)
        //.WithCredentials("username","password")
        .Build();
        //建立連線
        var session = cluster.Connect("benchmark");

        //準備 insert 資料用的 template
        var userttemplate = session.Prepare("INSERT INTO user (userid, name, birthday, currentsalary) VALUES (?, ?, ?, ?)");

        //將實際資料套入 insert 用的 template
        var statement = userttemplate.Bind(_user.UserId, _user.Name, _user.Birthday, _user.CurrentSalary);

        //實際執行指令
        await session.ExecuteAsync(statement);
        ```

    - Select

        ```cs
        //準備 cassandra 的連線資訊
        var cluster = Cluster.Builder()
        .AddContactPoints("127.0.0.1")
        .WithPort(9042)
        //.WithCredentials("username","password")
        .Build();
        //建立連線
        var session = cluster.Connect("benchmark");

        //初始化 mapper
        IMapper mapper = new Mapper(session);
        
        //從 cassandra 取得 user list
        IEnumerable<User> users = mapper.Fetch<User>("SELECT * FROM user");

        ```
    - Update
    
        ```cs
        //準備 cassandra 的連線資訊
        var cluster = Cluster.Builder()
        .AddContactPoints("127.0.0.1")
        .WithPort(9042)
        //.WithCredentials("username","password")
        .Build();
        //建立連線
        var session = cluster.Connect("benchmark");

        //準備 update 資料用的 template
        var userUpdateTemplate = session.Prepare("UPDATE benchmark.user SET name=? WHERE userid=?");

        //將實際資料套入 update 用的 template
        var statement = userUpdateTemplate.Bind("yowko",Guid.Parse("userid"));

        //實際執行指令
        await session.ExecuteAsync(statement);
        ```
    - Delete

        ```cs
        //準備 cassandra 的連線資訊
        var cluster = Cluster.Builder()
        .AddContactPoints("127.0.0.1")
        .WithPort(9042)
        //.WithCredentials("username","password")
        .Build();
        //建立連線
        var session = cluster.Connect("benchmark");

        //準備 delete 資料用的 template
        var userDeleteTemplate = session.Prepare("DELETE FROM benchmark.user WHERE userid = ?; ");

        //將實際資料套入 delete 用的 template
        var statement = userDeleteTemplate.Bind(Guid.Parse("useid"));

        //實際執行指令
        await session.ExecuteAsync(statement);
        ```

## 心得
在實際使用 Cassandra 前，我一直以為 Cassandra 與 MongoDB 一樣是 Document store 類型的 NoSQL，所以比照之前使用 MongoDB 經驗而有的先入為主印象：Cassandra 不需要預先定義欄位型態，沒想到 Cassandra 跟預期不同，這樣一來似乎就少了使用 Document store 類型 schema-free 的好處

實際使用之後 Cassanadra 特有的 CQL 與一般 SQL 相似性很高，非常方便，但思維上還需要做些調整：像是 user type、frozen 以及搜尋為導向的設計方式

# 參考資訊
1. [datastax/csharp-driver](https://github.com/datastax/csharp-driver)
2. [DataStax C# Driver for Apache Cassandra](https://docs.datastax.com/en/developer/csharp-driver/3.7/)
3. [Mapper component](https://docs.datastax.com/en/developer/csharp-driver/3.7/features/components/mapper/)