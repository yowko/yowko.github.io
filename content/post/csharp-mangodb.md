---
title: "使用 C# 存取 MongoDB"
date: 2019-02-16T22:40:00+08:00
lastmod: 2021-11-03T22:40:30+08:00
draft: false
tags: ["csharp","NoSQL","MongoDB"]
slug: "csharp-mangodb"
---
## 使用 C# 存取 MongoDB

之前筆記 [使用 C# 存取 Cassandra](https://blog.yokwo.com/csharp-cassandra) 提到想要將 log 存放至 NoSQL 中而正在嘗試某幾套 NoSQL，現在就來看看 MongoDB 的使用吧

## 基本環境說明

> 在 mac 上使用 docker 建立 MongoDB，接著使用 C# 連線至 MongoDB 執行基本 CRUD

1. macOS Mojave 10.14.2
2. Docker Community 18.09.1
3. Bogus 25.0.4
4. MongoDB 4.0.6
5. MongoDB.Driver 2.7.3

## 建立 MongoDB instance

> 透過 docker 僅建立單一 local MongoDB db instance

```bash
docker run -p 27017:27017 -d mongo
```

## 使用方式

1. 安裝 `MongoDB.Driver` NuGet 套件 : [mongodb/mongo-csharp-driver](https://github.com/mongodb/mongo-csharp-driver)

    > 本次測試使用 MongoDB.Driver 版本為 `2.7.3` ，另外 `mongocsharpdriver` 是相容於 MongoDB 1.X，官方已不建議在新專案上使用

    - Package Manager

        ```cmd
        Install-Package MongoDB.Driver
        ```

    - .NET CLI

        ```cmd
        dotnet add package MongoDB.Driver 
        ```

2.
3. 實際存取 MongoDB

    - Insert

        ```cs
        //準備 insert 用假資料
        var _user = GetFakeUserData();
        // mongodb 連線字串
        var connString = "mongodb://127.0.0.1:27017";
        //建立 mongo client
        var client = new MongoClient(connString);
        //取得 database
        var db = client.GetDatabase("benchmark");
        //取得 user collection
        var collection = db.GetCollection<User>("user");
        //針對 collection 插入單筆資料
        collection.InsertOneAsync(_user);
        ```

    - Select

        ```cs
        // mongodb 連線字串
        var connString = "mongodb://127.0.0.1:27017";
        //建立 mongo client
        var client = new MongoClient(connString);
        //取得 database
        var db = client.GetDatabase("benchmark");
        //取得 user collection
        var collection = db.GetCollection<User>("user");
        
        //依 name 過濾並取得一筆資料
        var document = collection.Find(a => a.Name == "name").FirstOrDefault();
        ```

    - Update

        ```cs
        // mongodb 連線字串
        var connString = "mongodb://127.0.0.1:27017";
        //建立 mongo client
        var client = new MongoClient(connString);
        //取得 database
        var db = client.GetDatabase("benchmark");
        //取得 user collection
        var collection = db.GetCollection<User>("user");
        
        // 設定過濾條件：找 Name 為 "name"
        var filter = Builders<User>.Filter.Eq(a => a.Name, "name");
        // 設定新的值
        var update = Builders<User>.Update.Set(a => a.Name, "Yowko");
        //將過濾條件與設定值傳入 collection 進行更新
        collection.UpdateOne(filter, update);
        ```

    - Delete

        ```cs
        // mongodb 連線字串
        var connString = "mongodb://127.0.0.1:27017";
        //建立 mongo client
        var client = new MongoClient(connString);
        //取得 database
        var db = client.GetDatabase("benchmark");
        //取得 user collection
        var collection = db.GetCollection<User>("user");
        // 設定過濾條件：找 Name 為 "name"
        var filter = Builders<User>.Filter.Eq(a => a.Name, "name");
        
        //刪除 Name 為 "Yowko" 的第一筆資料
        collection.DeleteOne(a => a.Name == "Yowko");
        //刪除符合 filter 條件的第一筆資料
        //collection.DeleteOne(filter);
        ```

## 心得

MongoDB 是老牌 Document 類型的 NoSQL 資料，在 Linux 與 Windows 都有不錯的支援程度，使用體驗也差不多

功能上也是目前用過幾套 NoSQL 中最豐富的，C# client 在多年發展下已顯得成熟，使用上比起過去方便許多，看來 MongoDB 受到青睞是有道理的

## 參考資訊

1. [MongoDB Driver Quick Tour](http://mongodb.github.io/mongo-csharp-driver/2.7/getting_started/quick_tour/)
2. [mongodb/mongo-csharp-driver](https://github.com/mongodb/mongo-csharp-driver)
