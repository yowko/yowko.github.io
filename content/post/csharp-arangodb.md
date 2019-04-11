---
title: "使用 C# 存取 ArangoDB"
date: 2019-02-12T22:40:00+08:00
lastmod: 2019-02-12T22:44:30+08:00
draft: false
tags: ["C#","NoSQL"]
slug: "csharp-arangodb"
---
# 使用 C# 存取 ArangoDB

之前筆記 [使用 C# 存取 Cassandra](https://blog.yokwo.com/csharp-cassandra) 提到想要將 log 存放至 NoSQL 中而正在嘗試某幾套 NoSQL，現在就來看看 ArangoDB 的使用吧

## 基本環境說明

> 在 mac 上使用 docker 建立 ArangoDB，接著使用 C# 連線至 ArangoDB 執行基本 CRUD

1. macOS Mojave 10.14.2
2. Docker Community 18.09.1
3. Bogus 25.0.4
4. ArangoDB 3.4.2
5. ArangoDB.Client 0.7.70

## 建立 ArangoDB instance

> 透過 docker 僅建立單一 local ArangoDB db instance，並指定 `root` 的 password 為 `pass.123`

```bash
docker run -e ARANGO_ROOT_PASSWORD=pass.123 -p 8529:8529 -d arangodb
```

## 使用方式

> 透過 http://localhost:8529 可以開啟設定介面

1. 登入 ArangoDB

    ![1login](https://user-images.githubusercontent.com/3851540/52642842-bdb43280-2f16-11e9-881e-9a62c63a98da.png)

    ![2selectdb](https://user-images.githubusercontent.com/3851540/52642845-be4cc900-2f16-11e9-8b4f-e303a3703a39.png)
2. 建立 Database

    ![3createdb](https://user-images.githubusercontent.com/3851540/52642846-be4cc900-2f16-11e9-9d2d-1c40418c10aa.png)

    ![4createdb2](https://user-images.githubusercontent.com/3851540/52642847-be4cc900-2f16-11e9-972c-0628f4f769d1.png)

3. 建立 Collection

    ![5createcollection](https://user-images.githubusercontent.com/3851540/52642848-bee55f80-2f16-11e9-9be4-d59fb3df8400.png)

    ![6createcollection2](https://user-images.githubusercontent.com/3851540/52642849-bee55f80-2f16-11e9-833b-9d3be0430e4c.png)

4. 安裝 NuGet 套件：
    - Package Manager

        ```
        Install-Package ArangoDB.Client
        ``` 
    - .NET CLI

        ```
        dotnet add package ArangoDB.Client
        ```
5. 實際存取 ArangoDB

    - Insert

        ```cs
        //準備 insert 用假資料
        var _user = GetFakeUserData();

        ArangoDatabase.ChangeSetting(s =>
        {
            s.Database = "benchmark";//指定 database
            s.Url = "http://localhost:8529";// 指定 Arango url
        
            s.Credential = new NetworkCredential("root", "pass.123");//指定 credential
        });

        using (var db = ArangoDatabase.CreateWithSetting())
        {
            //將資料寫入 "User" collection 中
            db.Collection("User").Insert(_user);
        }
        ```

    - Select

        ```cs
        ArangoDatabase.ChangeSetting(s =>
        {
            s.Database = "benchmark";//指定 database
            s.Url = "http://localhost:8529";// 指定 Arango url
        
            s.Credential = new NetworkCredential("root", "pass.123");//指定 credential
        });

        using (var db = ArangoDatabase.CreateWithSetting())
        {
            var user = db.Document<User>"document-key");
        }
        ```
    
    - Update
    
        ```cs
        ArangoDatabase.ChangeSetting(s =>
        {
            s.Database = "benchmark";//指定 database
            s.Url = "http://localhost:8529";// 指定 Arango url
        
            s.Credential = new NetworkCredential("root", "pass.123");//指定 credential
        });

        using (var db = ArangoDatabase.CreateWithSetting())
        {
            //依 document-key 修改 Name 欄位資料內容為 "Yowko"
            db.UpdateById<User>("document-key", new { Name = "Yowko" });    
        }
        ```
    - Delete

        ```cs
        ArangoDatabase.ChangeSetting(s =>
        {
            s.Database = "benchmark";//指定 database
            s.Url = "http://localhost:8529";// 指定 Arango url
        
            s.Credential = new NetworkCredential("root", "pass.123");//指定 credential
        });

        using (var db = ArangoDatabase.CreateWithSetting())
        {
            //依 document-key 刪除資料
        	db.RemoveById<User>("document-key");
        }
        ```

## 心得
ArangoDB 與 RavenDB、CouchDB 相同預設就帶有 GUI 管理介面不用另外準備，管理上很便利

可以滿足 Shcema-less 的需求，資料的處理上可以透過 AQL 或是 REST 方式來操作，.NET 使用的套件：ArangoDB.Client 底層也是也是直接使用 Http 來發送 request ，不過在 api 的使用上比起 RavenDB.Client 便利性稍差 


# 參考資訊
1. [ra0o0f/arangoclient.net](https://github.com/ra0o0f/arangoclient.net)
2. [Tutorial: C# in 10 Minutes](https://github.com/ra0o0f/arangoclient.net/blob/next/docs/tutorial/csharp-in-10-minutes.md)