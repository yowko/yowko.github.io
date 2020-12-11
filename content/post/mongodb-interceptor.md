---
title: "取得 MongoDB SDK 實際產生的指令"
date: 2019-08-03T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["C#","MongoDB"]
slug: "mongodb-interceptor"
---

## 取得 MongoDB SDK 實際產生的指令

之前曾經在 [取得 Entity Framework 存取 DB 時的實際 SQL Script](/entityframework-log-sql/) 提到如果都透過 EntityFramework 來存取 DB，有時候遇到的效能瓶頸是因為 EntityFramework 沒有使用正確的 index 或是執行計劃，這時候就可以使用 interceptor 來攔截 EntityFramework 實際對 DB 執行的 SQL statement 內容

最近剛好要確認專案的 MongoDB instance performane issue，剛好趁這個機會紀錄一下如何取得 MongoDB SDK 實際執行的 script 內容

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. MongoDB 4.0.10
4. NuGet Package

    - MongoDB.Driver 2.8.1
    - Bogus 28.0.2

        > 產生假資料用，選擇性安裝

5. 測試用程式

    - 連線

        ```cs
        var client = new MongoClient(
            new MongoClientSettings
            {
               Server = new MongoServerAddress("localhost",27017)
            });
        var db = client.GetDatabase("test");
        var collection = db.GetCollection<User>("users");

        //準備 insert 用假資料
        var _user = GetFakeUserData();
        ```

    - insert

        ```cs
        await collection.InsertOneAsync(_user);
        ```

    - select

        ```cs
        //依 name 過濾並取得一筆資料
        var document = collection.Find(a => a.Name == "name").FirstOrDefault();
        ```

    - update

        ```cs
        // 設定過濾條件：找 Name 為 "name"
        var filter = Builders<User>.Filter.Eq(a => a.Name, "name");
        // 設定新的值
        var update = Builders<User>.Update.Set(a => a.Name, "Yowko");
        //將過濾條件與設定值傳入 collection 進行更新
        collection.UpdateOne(filter, update);
        ```

    - delete

        ```cs
        collection.DeleteOne(a => a.Name == "Yowko");
        ```

    - upsert

        ```cs
        var upsertModel = Builders<User>.Update
            .SetOnInsert(m => m.Id, _user.Id)
            .Set(m => m.ModifiedOn, _user.ModifiedOn)
            .SetOnInsert(m => m.CreatedOn, _user.CreatedOn)
            .Set(m => m.Name, _user.Name);
        var result = await db
            .GetCollection<User>("users")
            .UpdateOneAsync(m => m.Id == _user.Id, upsertModel,
                new UpdateOptions {IsUpsert = true}).ConfigureAwait(false);
        ```

## 指令取得方式

1. 修改 Mongo client 設定

    >加入訂閱 `CommandStartedEvent`

    ```cs
    var client = new MongoClient(
        new MongoClientSettings
        {
            Server = new MongoServerAddress("localhost", 27017),
            ClusterConfigurator = cb =>
            {
                cb.Subscribe<CommandStartedEvent>(e =>
                {
                    Console.WriteLine($"{e.CommandName} : {e.Command}");
                });
            }
        });
    ```

2. 實際效果

    ![1subscribe](https://user-images.githubusercontent.com/3851540/62420447-1940cb00-b6c5-11e9-90e0-8381ff79c85b.png)

## 心得

MongoDB .NET driver 曾經在 2.0 時短暫加入 logging 相關的實驗性功能：可以傳入自訂的 listener 來接收 log，但後來的版本就移除了，讓我在筆記時一度懷疑起是不是自己記錯 XD

紀錄實際產生的 script 內容，在 MongoDB 使用量大的系統中可能造成 log 數量龐大的問題，但類似情況在其他類型的 log 也會遇到 - 如果在系統開發階段沒有加入需要的 log 內容跟設定，或是 production 環境沒有開啟對應的 log level，到時候出現問題時可能就會遇到要追查 issue 但相關材料不足的情況

## 參考資訊

1. [取得 Entity Framework 存取 DB 時的實際 SQL Script](/entityframework-log-sql/)
2. [MongoDB C# Driver: Query interceptors?](https://stackoverflow.com/questions/48947260/mongodb-c-sharp-driver-query-interceptors)
3. [How do I log my queries in MongoDB C# Driver 2.0?](https://stackoverflow.com/questions/30333925/how-do-i-log-my-queries-in-mongodb-c-sharp-driver-2-0)
4. [What’s New in the MongoDB .NET 2.0 Driver](http://mongodb.github.io/mongo-csharp-driver/2.0/what_is_new/#logging)
