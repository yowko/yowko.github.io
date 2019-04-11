---
title: "使用 C# 存取 CouchDB"
date: 2019-02-10T23:43:00+08:00
lastmod: 2019-02-10T23:44:30+08:00
draft: false
tags: ["C#","NoSQL"]
slug: "csharp-couchdb"
---
# 使用 C# 存取 CouchDB

之前筆記 [使用 C# 存取 Cassandra](https://blog.yokwo.com/csharp-cassandra) 提到想要將 log 存放至 NoSQL 中而正在嘗試某幾套 NoSQL，現在就來看看 CouchDB 的使用吧

## 基本環境說明

> 在 mac 上使用 docker 建立 CouchDB，接著使用 C# 連線至 CouchDB 執行基本 CRUD

1. macOS Mojave 10.14.2
2. Docker Community 18.09.1
3. Bogus 25.0.4
4. CouchDB 2.3.0
5. MyCouch 6.0.0

## 建立 CouchDB instance

> 透過 docker 僅建立單一 local CouchDB db instance

```bash
docker run -p 5984:5984 -d couchdb
```

## 使用方式

> 透過 http://127.0.0.1:5984/_utils 可以開啟設定介面

1. 建立 Database

    ![1createdb](https://user-images.githubusercontent.com/3851540/52535036-3fc02200-2d84-11e9-86e0-644dc8ecb747.png)

2. 安裝 NuGet 套件：
    - Package Manager

        ```
        Install-Package MyCouch
        ``` 
    - .NET CLI

        ```
        dotnet add package MyCouch
        ```
3. 實際存取 CouchDB

    - Insert

        ```cs
        //準備 insert 用假資料
        var _user = GetFakeUserData();
        
        //連線至 CouchDB 的 benchmark Database
        using (var client = new MyCouchClient("http://localhost:5984/", "benchmark"))
        {
            //新增資料
            await client.Documents.PostAsync(JsonConvert.SerializeObject(_user));
        }
        ```

    - Select

        ```cs
        //連線至 CouchDB 的 benchmark Database
        using (var client = new MyCouchClient("http://localhost:5984/", "benchmark"))
        {
            //依 id 從 CouchDB 中取得資料
            await client.Documents.GetAsync("id");
        }
        ```
    - Update
    
        ```cs
        //連線至 CouchDB 的 benchmark Database
        using (var client = new MyCouchClient("http://localhost:5984/", "benchmark"))
        {
            // 指定 id 與 docRevision 來更新內容            
            await client.Documents.PutAsync("id","docRevision", JsonConvert.SerializeObject(new {name="yowko"}));
        }
        ```
    - Delete

        ```cs
        //連線至 CouchDB 的 benchmark Database
        using (var client = new MyCouchClient("http://localhost:5984/", "benchmark"))
        {
            //依 id 與 docRevision 刪除檔案
            await client.Documents.DeleteAsync("id","docRevision");
        }
        ```

## 心得
CouchDB 預設就帶有 GUI 管理介面不用另外準備，管理上很便利，另外是 API 部份 CouchDB 都是直接使用 Http 來發送 request 所以 API 的命名也很有 Http RESTful 風格，但直覺上透過 Http 應該沒辦法快速處理大量資料，就待後續效能比較結果囉


# 參考資訊
1. [danielwertheim/mycouch](https://github.com/danielwertheim/mycouch)
2. [Apache CouchDB 2.3 Documentation](http://docs.couchdb.org/en/stable/index.html)