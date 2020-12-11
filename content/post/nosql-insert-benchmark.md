---
title: "[Benchmark] 使用 C# 對 NoSQL insert 操作的效能數據"
date: 2019-02-24T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["C#","MongoDB","Cassandra","NoSQL"]
slug: "nosql-insert-benchmark"
---
# [Benchmark] 使用 C# 對 NoSQL insert 操作的效能數據
最近專案需要將收到的原始 request 內容直接儲存下來，以備日後有問題或是後續加工使用。

針對這類只有 insert 跟 select 操作的需要，過去大多透過 MongoDB 做為儲存媒介，但幾次使用下來對於 c# 操作 MongoDB 的 api 始終覺得不太順手，加上後來陸續看了些效能比較的數據， MongoDB 似乎沒有佔到優勢，所以想趁著這次機會試用 Cassandra , PostgreSQL , ArangoDB , RavenDB , CouchDB ，不過想要說服自己或是團隊改用其他技術最重要的莫過於執行效率了，於是老話一句：天下武功唯快不破，就直接來看看模擬實際情境透過 c# 來進行 insert data 的效能數據吧

因為這次開發的系統目標 concurrent user 及操作量很大，所以可以預見 log 數量也會有相當的級別，因此想要先測試 insert 的效能，先扛得住寫入再來想後續讀取的問題

## 基本環境說明
1. macOS Mojave 10.14.2
2. intel i5 2.3G
3. 16 GB 2133 MHz Ram
4. 各 NoSQL 皆使用官方最新版 (lastest) docker image 建立一個 node
    - MongoDB 4.0.6
    - Cassandra 3.11.3
    - PostgreSQL 11.2
    - ArangoDB 3.4.2
    - RavenDB 4.1
    - CouchDB 2.3.0
5. .NET Core 2.2.101
6. Bogus 25.0.4
7. BenchmarkDotNet 0.11.4
8. 驗證用 data poco

    ```cs
    class User
    {
        public Guid UserId { get; set; }
        public string Name { get; set; }
        public DateTime Birthday { get; set; }
        public decimal CurrentSalary { get; set; }
    }
    ```
9. Bogus 套件製造假資料

    ```cs
    User GetFakeUserData()
    {
        var fakesUsers = new Faker<User>()
                    .RuleFor(a => a.UserId, f => f.Random.Guid())
                    .RuleFor(a => a.Name, f => f.Name.FirstName())
                    .RuleFor(a => a.Birthday, f => f.Date.Past())
                    .RuleFor(a => a.CurrentSalary, f => f.Random.Decimal(0M, 10,000M))
                    .Generate();

        return fakesUsers;
    }
    ```

## NoSQL 測試語法
### MongoDB

1. 環境建立流程及使用可以參考 [使用 C# 存取 MongoDB](/csharp-mangodb/)

2. Insert 語法

    ```cs
    var client = new MongoClient();
    var db = client.GetDatabase("benchmark");
    var collection = db.GetCollection<User>("users");
    var fakesUsers = UserUtility.GetFakeUsers(Times);

    await collection.InsertManyAsync(fakesUsers);
    ```
3. 數據

    |        Method |   Times |          Mean |      Error |     StdDev |        Median |
    |-------------- |-------- |--------------:|-----------:|-----------:|--------------:|
    | MongoDbInsert |     100 |      4.824 ms |  0.2087 ms |  0.6087 ms |      4.648 ms |
    | MongoDbInsert |   10,000 |    111.686 ms |  2.2250 ms |  2.3807 ms |    111.603 ms |
    | MongoDbInsert | 1,000,000 | 10,725.614 ms | 65.2694 ms | 61.0531 ms | 10,727.612 ms |


### Cassandra

1. 環境建立流程及使用可以參考 [使用 C# 存取 Cassandra](/csharp-cassandra/)

2. Insert 語法

    ```cs
    var cluster = Cluster.Builder()
                .AddContactPoints("127.0.0.1")
                .WithPort(9042)
                .Build();

    var session = cluster.Connect("benchmark");

    var userttemplate = session.Prepare("INSERT INTO users (userid, name, birthday, currentsalary) VALUES (?, ?, ?, ?)");


    foreach (var userList in UserUtility.SpiltBySize(UserUtility.GetFakeUsers(Times), 100))
    {
        var batch = new BatchStatement();
        foreach (var _user in userList)
        {
            batch.Add(userttemplate.Bind(_user.UserId, _user.Name, _user.Birthday, _user.CurrentSalary));
        }

        session.Execute(batch);
    }
    ```
3. 數據

    |          Method |   Times |          Mean |        Error |       StdDev |
    |---------------- |-------- |--------------:|-------------:|-------------:|
    | CassandraInsert |     100 |      37.53 ms |     2.047 ms |     6.004 ms |
    | CassandraInsert |   10,000 |   1,633.61 ms |    33.614 ms |    97.519 ms |
    | CassandraInsert | 1,000,000 | 165,795.11 ms | 3,301.669 ms | 4,293.102 ms |

### CouchDB

1. 環境建立流程及使用可以參考 [使用 C# 存取 CouchDB](/csharp-couchdb/)

2. Insert 語法

    ```cs
    var connectionString = "http://localhost:5984/";
    var client = new CouchClient(connectionString);
    var db = await client.GetDatabaseAsync("benchmark");
    var fakesUsers = UserUtility.GetFakeUsers(Times);
    foreach (var userList in UserUtility.SpiltBySize(fakesUsers, 10,000))
    {
        await db.BulkInsertAsync(userList.ToArray());
    }
    ```
3. 數據

    |     Method |   Times |          Mean |         Error |       StdDev |
    |----------- |-------- |--------------:|--------------:|-------------:|
    | InsertTest |     100 |      36.62 ms |     0.7554 ms |     2.131 ms |
    | InsertTest |   10,000 |   1,628.92 ms |    26.8867 ms |    25.150 ms |
    | InsertTest | 1,000,000 | 174,499.33 ms | 2,222.5798 ms | 1,970.259 ms |

### RavenDB

1. 環境建立流程及使用可以參考 [使用 C# 存取 RavenDB](/csharp-ravendb/)

2. Insert 語法

    ```cs
    var store = new DocumentStore
            {
                Urls = new string[] {"http://localhost:8080"},
            };
    store.Initialize();
    var fakeuserUSers = UserUtility.GetFakeUsers(Times);
    using (BulkInsertOperation bulkInsert = store.BulkInsert("benchmark"))
    {
        foreach (var user in fakeuserUSers)
        {
            bulkInsert.Store(user);
        }
    }
    ```
3. 數據

    |     Method |   Times |          Mean |        Error |     StdDev |
    |----------- |-------- |--------------:|-------------:|-----------:|
    | InsertTest |     100 |      44.04 ms |     1.477 ms |   4.332 ms |
    | InsertTest |   10,000 |   1,066.53 ms |    21.480 ms |  43.877 ms |
    | InsertTest | 1,000,000 | 102,426.60 ms | 1,157.901 ms | 966.900 ms |


### ArangoDB

1. 環境建立流程及使用可以參考 [使用 C# 存取 ArangoDB](/csharp-arangodb/)

2. Insert 語法

    ```cs
    ArangoDatabase.ChangeSetting(s =>
    {
        s.Database = "benchmark";
        s.Url = "http://localhost:8529";

        s.Credential = new NetworkCredential("root", "pass.123");
        s.SystemDatabaseCredential = new NetworkCredential("root", "pass.123");
    });
    var fakesUsers = UserUtility.GetFakeUsers(Times);
    using (var db = ArangoDatabase.CreateWithSetting())
    {
        foreach (var userList in UserUtility.SpiltBySize(fakesUsers, 10,000))
        {
            db.Collection("Users").InsertMultiple(userList);
        }
    }
    ```
3. 數據

    |     Method |   Times |      Mean |     Error |    StdDev |    Median |
    |----------- |-------- |----------:|----------:|----------:|----------:|
    | InsertTest |     100 |  15.24 ms |  1.369 ms |  3.927 ms |  14.02 ms |
    | InsertTest |   10,000 | 696.75 ms | 13.726 ms | 28.652 ms | 692.08 ms |
    | InsertTest | 1,000,000 |        NA |        NA |        NA |        NA |


### PostgreSQL

1. 環境建立流程及使用可以參考 [使用 C# 存取 PostgreSQL](/csharp-postgresql/)

2. Insert 語法

    ```cs
    string connString = "Server=127.0.0.1; User Id=postgres; Database=benchmark; Port=5432; Password=pass.123; SSL Mode=Prefer; Trust Server Certificate=true";
    var fakeuserUSers = UserUtility.GetFakeUsers(Times).Select(a=>JsonConvert.SerializeObject(a));
    using (var conn = new NpgsqlConnection(connString))
    {
        conn.Open();
        using (var writer = conn.BeginBinaryImport( "COPY test.users (\"User\") FROM STDIN (FORMAT BINARY)"))
        {
            foreach (var user in fakeuserUSers)
            {
                writer.StartRow();
                writer.Write(user, NpgsqlDbType.Json);
            }
            writer.Complete();
        }
    }
    ```
3. 數據

    |     Method |   Times |         Mean |       Error |      StdDev |       Median |
    |----------- |-------- |-------------:|------------:|------------:|-------------:|
    | InsertTest |     100 |     6.501 ms |   0.2907 ms |   0.8386 ms |     6.256 ms |
    | InsertTest |   10,000 |    88.128 ms |   2.6015 ms |   6.8987 ms |    86.356 ms |
    | InsertTest | 1,000,000 | 7,523.930 ms | 147.6126 ms | 220.9395 ms | 7,540.428 ms |

## 實際數據比較
1. 100

    |     Method |   Mean |
    |----------- |-------- |
    |MongoDB|<font style="color:lightgreen">4.824 ms</font>|
    |Cassandra|37.53 ms|
    |CouchDB|36.62 ms|
    |RavenDB|<font style="color:red">44.04 ms</font>|
    |ArangoDB|15.24 ms|
    |PostgreSQL- json|6.501 ms|

2. 10,000

    |     Method |   Mean |
    |----------- |-------- |
    |MongoDB|111.686 ms |
    |Cassandra|<font style="color:red">1,633.61 ms</font>|
    |CouchDB|1,628.92 ms|
    |RavenDB|1,066.53 ms|
    |ArangoDB|696.75 ms|
    |PostgreSQL -json|<font style="color:lightgreen">88.128 ms</font>|

3. 1,000,000

    |     Method |   Mean |
    |----------- |-------- |
    |MongoDB|10,725.614 ms|
    |Cassandra|165,795.11 ms|
    |CouchDB|<font style="color:red">174,499.33 ms</font>|
    |RavenDB|102,426.60 ms|
    |ArangoDB|<font style="color:red">NA</font>|
    |PostgreSQL -json|<font style="color:lightgreen">7,523.930 ms</font>|


## 心得

為了選擇 NoSQL ，事前查過網路上的相關比較：[Cassandra vs. MongoDB vs. Couchbase vs. HBase](https://www.datastax.com/nosql-databases/benchmarks-cassandra-vs-mongodb-vs-hbase) 與 [NoSQL Performance Benchmark 2018: MongoDB, PostgreSQL, OrientDB, Neo4j and ArangoDB](https://dzone.com/articles/nosql-performance-benchmark-2018-mongodb-postgresq)，就相關文獻看來選擇 Cassandra 的機會相對較高，因此我們確實也使用 Cassandra 完成了 POC，但 POC 過程中發現 .NET client 有一些缺陷，其中最嚴重的幾個大資料操作 API：CQLSSTableWriter、sstableloader 並沒有實作，讓實際透過 C# 操作 Cassandra 時效能並不如宣稱中的優異

另外 ArangoDB 在中量資料數時表現並不差，只是在百萬筆資料的測試情境一直無法完成測試(已嘗試拆為幾個小 batch)，較為可惜

反而是 MongoDB 不愧是最受歡迎的 document 類型 NoSQL，效能表現非常令人讚賞，但最令人驚豔的就屬 PostgreSQL ，以 RDBMS 之姿在 json insert 的操作上竟打敗其他 NoSQL ，實在太強大了

關於這次我堅持自行測試效能數據，我認為團隊應該使用自己熟悉的語言、環境、工具來進行 POC，便可以儘早反應出實際使用上的現象，以本次測試為例：datasax 跑出的 Cassandra 數據完全碾壓 MongoDB 但加上語言及 library 因素後結果卻完全不同

### 2019-03-01 補充
以測試結果來看，PostgreSQL 的 insert 數據是最好的，但該使用哪套技術則不是單純由數據來可以下定論的，以我所在團隊為例：雖然知道 PostgreSQL 在 insert json 的速度最快，但最後選了第二名的 MongoDB, 主要是團隊多數成員都有 MongoDB 使用經驗，對於 PostgreSQL 則幾乎沒有，更遑論 PostgreSQL 的調校、HA 機制架設能力 


# 參考資料
1. [Cassandra vs. MongoDB vs. Couchbase vs. HBase](https://www.datastax.com/nosql-databases/benchmarks-cassandra-vs-mongodb-vs-hbase)
2. [NoSQL Performance Benchmark 2018: MongoDB, PostgreSQL, OrientDB, Neo4j and ArangoDB](https://dzone.com/articles/nosql-performance-benchmark-2018-mongodb-postgresq)