---
title: "使用 C# 存取 RavenDB"
date: 2019-02-11T22:45:00+08:00
lastmod: 2021-11-03T22:44:30+08:00
draft: false
tags: ["csharp","NoSQL"]
slug: "csharp-ravendb"
---
## 使用 C# 存取 RavenDB

之前筆記 [使用 C# 存取 Cassandra](https://blog.yokwo.com/csharp-cassandra) 提到想要將 log 存放至 NoSQL 中而正在嘗試某幾套 NoSQL，現在就來看看 RavenDB 的使用吧

## 基本環境說明

> 在 mac 上使用 docker 建立 RavenDB，接著使用 C# 連線至 RavenDB 執行基本 CRUD

1. macOS Mojave 10.14.2
2. Docker Community 18.09.1
3. Bogus 25.0.4
4. RavenDB 4.1
5. RavenDB.Client 4.1.3

## 建立 RavenDB instance

> 透過 docker 僅建立單一 local RavenDB db instance

```bash
docker run -d -p 8080:8080 ravendb/ravendb
```

## 使用方式

> 透過 <http://localhost:8080/> 可以開啟設定介面

1. 設定 RavenDB

    ![1createravendb](https://user-images.githubusercontent.com/3851540/52570619-c3dedc00-2e4e-11e9-9391-76f26ed207e0.png)
2. 建立 Database

    ![2createdb](https://user-images.githubusercontent.com/3851540/52570614-c3464580-2e4e-11e9-91d1-0e03264d51bd.png)

    ![3createdb2](https://user-images.githubusercontent.com/3851540/52570615-c3dedc00-2e4e-11e9-858c-e5353c2a1e1e.png)

3. 安裝 NuGet 套件：
    - Package Manager

        ```cmd
        Install-Package RavenDB.Client
        ```

    - .NET CLI

        ```cmd
        dotnet add package RavenDB.Client
        ```

4. 實際存取 RavenDB

    - Insert

        ```cs
        //準備 insert 用假資料
        var _user = GetFakeUserData();

        var store = new DocumentStore
        {
            // 指定所有 RavenDB cluster url
            Urls = new string[] { "http://localhost:8080" },
            // 指定 Database
            Database = "benchmark",    
        };
        //初始化 DocumentStore 
        store.Initialize();

        // 開啟 RavenDB session 
        using (IDocumentSession session = store.OpenSession()) 
        {
            // 建立 collection 並開始 track entity
            session.Store(_user);
            //將資料儲存至 db
            session.SaveChanges();
        }
        ```

    - Select

        ```cs
        var store = new DocumentStore
        {
            // 指定所有 RavenDB cluster url
            Urls = new string[] { "http://localhost:8080" },
            // 指定 Database
            Database = "benchmark",    
        };
        //初始化 DocumentStore 
        store.Initialize();

        // 開啟 RavenDB session 
        using (IDocumentSession session = store.OpenSession()) 
        {
            List<User> users = session
            .Query<User>()
            .ToList();
        }
        ```

    - Update

        ```cs
        var store = new DocumentStore
        {
            // 指定所有 RavenDB cluster url
            Urls = new string[] { "http://localhost:8080" },
            // 指定 Database
            Database = "benchmark",    
        };
        //初始化 DocumentStore 
        store.Initialize();

        // 開啟 RavenDB session 
        using (IDocumentSession session = store.OpenSession()) 
        {
            //從 RavenDB 取得單筆資料
            User user = session
            .Query<User>()
            .FirstOrDefault(u => u.UserId== Guid.Parse("userid"));
            
            user.Name="yowko";
            //將資料儲存至 db
            session.SaveChanges();
        }
        ```

    - Delete

        ```cs
        var store = new DocumentStore
        {
            // 指定所有 RavenDB cluster url
            Urls = new string[] { "http://localhost:8080" },
            // 指定 Database
            Database = "benchmark",    
        };
        //初始化 DocumentStore 
        store.Initialize();

        // 開啟 RavenDB session 
        using (IDocumentSession session = store.OpenSession()) 
        {
            session.Delete("collection/id");
            session.SaveChanges();
        }
        ```

## 心得

RavenDB 與 CouchDB 相同預設就帶有 GUI 管理介面不用另外準備，管理上很便利

可以滿足 Shcema-less 的需求，RavenDB.Client 在 DB 的操作上底層也是也是直接使用 Http 來發送 request ，不過在 api 的使用上比起 CouchDB 對於 LINQ 支援度更高，非常符合 .NET 開發人員的使用習慣

## 參考資訊

1. [ravendb/ravendb](https://github.com/ravendb/ravendb)
2. [RavenDB Documentation](https://ravendb.net/docs/article-page/4.1/csharp)
