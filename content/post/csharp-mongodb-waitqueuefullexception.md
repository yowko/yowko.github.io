---
title: ".Net Core 操作 MongoDB 出現 MongoWaitQueueFullException"
date: 2019-07-28T21:30:00+08:00
lastmod: 2019-07-28T21:30:31+08:00
draft: false
tags: ["C#","dotnet core","MongoDB"]
slug: "csharp-mongodb-waitqueuefullexception"
---

## .Net Core 操作 MongoDB 出現 MongoWaitQueueFullException

之前筆記 [在 Docker Compose file 3 下限制 CPU 與 Memory](https://blog.yowko.com/docker-compose-3-cpu-memory-limit/) 中提到同事反應說某個專案在執行時會造成 MongoDB CPU high，最後引發 docker service crash，一直沒有找到 root cause，想說趁著假日有完整時間可以仔細 debug，想不到一開始就遇到意料外的錯誤，就來看看這預期之外的錯誤該如何解決吧

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core 2.2.301
3. MongoDB 4.0.10
4. NuGet Package

    - MongoDB.Driver 2.8.1

5. 測試用程式碼

    - Model

        ```cs
        [BsonIgnoreExtraElements]
        public class User
        {
            [BsonId]
            public int Id { get; set; }
            public string Name { get; set; }
            public DateTime CreatedOn { get; set; }
            public DateTime ModifiedOn { get; set; }
        }
        ```

    - Method

        ```cs
        var client = new MongoClient(
            new MongoClientSettings
            {
               Server = new MongoServerAddress("localhost",27017),
            });
        var db = client.GetDatabase("test");

        Parallel.For(0, 100000,new ParallelOptions { MaxDegreeOfParallelism = 2 },async (i) =>
        {
            var user = new Faker<User>()
                .RuleFor(a => a.Id, f => f.Random.Int())
                .RuleFor(a => a.Name, f => f.Name.FullName())
                .RuleFor(a => a.CreatedOn, DateTime.UtcNow)
                .RuleFor(a => a.ModifiedOn, DateTime.UtcNow).Generate();

            var upsertModel = Builders<User>.Update
                .SetOnInsert(m => m.Id, user.Id)
                .Set(m => m.ModifiedOn, user.ModifiedOn)
                .SetOnInsert(m => m.CreatedOn, user.CreatedOn)
                .Set(m => m.Name, user.Name);
            var result = await db
                .GetCollection<User>("users")
                .UpdateOneAsync(m => m.Id == user.Id, upsertModel,
                    new UpdateOptions {IsUpsert = true}).ConfigureAwait(false);

            Console.WriteLine(result.IsAcknowledged);
        });
        ```

## 錯誤訊息

1. 訊息內容

    ```txt
    Unhandled Exception: MongoDB.Driver.MongoWaitQueueFullException: The wait queue for server selection is full.
    at MongoDB.Driver.Core.Clusters.Cluster.EnterServerSelectionWaitQueue()
    at MongoDB.Driver.Core.Clusters.Cluster.SelectServerHelper.WaitingForDescriptionToChange()
    at MongoDB.Driver.Core.Clusters.Cluster.SelectServerAsync(IServerSelector selector, CancellationToken cancellationToken)
    at MongoDB.Driver.MongoClient.AreSessionsSupportedAfterSeverSelctionAsync(CancellationToken cancellationToken)
    at MongoDB.Driver.MongoClient.AreSessionsSupportedAsync(CancellationToken cancellationToken)
    at MongoDB.Driver.MongoClient.StartImplicitSessionAsync(CancellationToken cancellationToken)
    at MongoDB.Driver.MongoCollectionImpl`1.UsingImplicitSessionAsync[TResult](Func`2 funcAsync, CancellationToken cancellationToken)
    at MongoDB.Driver.MongoCollectionBase`1.UpdateOneAsync(FilterDefinition`1 filter, UpdateDefinition`1 update, UpdateOptions options, Func`3 bulkWriteAsync)
    at TestMongo.Program.<>c__DisplayClass0_0.<<Main>b__0>d.MoveNext() in /Users/yowko.tsai/TestMongo/TestMongo/Program.cs:line 38
    --- End of stack trace from previous location where exception was thrown ---
    at System.Threading.ExecutionContext.RunInternal(ExecutionCon
    ```

2. 錯誤截圖

    ![1exception](https://user-images.githubusercontent.com/3851540/62009104-e1e69180-b18d-11e9-97ae-54dd21c28fb0.png)

## 問題發生原因

從測試程式碼來看，只建立了一個 connection instance，不是 hit 到 MongoDB connection pool 預設上限 ( `maxConnectionPoolSize` 預設值是 `100`) 的問題，但因 connection 僅有一條，因此所有操作都需要等待 connection 當前操作結束後才能取得所有權來執行指令，所以其他的 thread 就會被放入 waitQueue 中，而預設 waitQueue 大小為 `500` = `5` (waitQueueMultiple) * `100` (maxConnectionPoolSize)，在平行執行下快速累積 waitQueue 內容最後超出 default WaitQueueSize 而拋出錯誤

## 解決方式

> 下列方式皆可避免 `MongoWaitQueueFullException`，擇一或是搭配使用皆可

1. 增加 `WaitQueueSize`

    > 實際數量則需依執行的目標數來決定

    ```cs
    var client = new MongoClient(
    new MongoClientSettings
    {
       Server = new MongoServerAddress("localhost",27017) ,
       WaitQueueSize = 20000
    });
    ```

2. 限制平行程度

    ```cs
    Parallel.For(0, 100000,new ParallelOptions { MaxDegreeOfParallelism = 2 },async (i) =>
    {
    }
    ```

    或是直接改用 `for`

    ```cs
    for (var i = 0; i < 100000; i++)
    {
       var user = new Faker<User>()
           .RuleFor(a => a.Id, f => f.Random.Int())
           .RuleFor(a => a.Name, f => f.Name.FullName())
           .RuleFor(a => a.CreatedOn, DateTime.UtcNow)
           .RuleFor(a => a.ModifiedOn, DateTime.UtcNow).Generate();

       var upsertModel = Builders<User>.Update
           .SetOnInsert(m => m.Id, user.Id)
           .Set(m => m.ModifiedOn, user.ModifiedOn)
           .SetOnInsert(m => m.CreatedOn, user.CreatedOn)
           .Set(m => m.Name, user.Name);
       var result = await db
           .GetCollection<User>("users")
           .UpdateOneAsync(m => m.Id == user.Id, upsertModel,
               new UpdateOptions {IsUpsert = true}).ConfigureAwait(false);
       Console.WriteLine(result.IsAcknowledged);
    }
    ```

## 心得

一直以來我對 MongoDB 就有種又愛又恨的心情，常常踩中些莫名奇妙的坑 (我知道是我自己沒有好好花時間徹底理解它造成的)，但它的效能以及 c# 使用的方便性又讓我很難放棄它 QQ

每次踩完坑都僥倖覺得下次應該不會再用 MongoDB，也就沒留下什麼文件跟筆記，但事實上卻是老是在查相似的東西，真的一點長進都沒有 @@"

## 參考資訊

1. [The wait queue for acquiring a connection to server is full](https://jira.mongodb.org/browse/CSHARP-1180?focusedCommentId=1291455&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-1291455)
2. [MongoWaitQueueFullException: The wait queue for acquiring a connection to server is full](https://stackoverflow.com/questions/37322110/mongowaitqueuefullexception-the-wait-queue-for-acquiring-a-connection-to-server)
