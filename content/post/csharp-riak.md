---
title: "使用 C# 存取 Riak"
date: 2019-02-23T20:40:00+08:00
lastmod: 2021-11-03T20:40:30+08:00
draft: false
tags: ["csharp","NoSQL"]
slug: "csharp-riak"
---
## 使用 C# 存取 Riak

之前筆記 [使用 C# 存取 Cassandra](https://blog.yokwo.com/csharp-cassandra) 提到想要將 log 存放至 NoSQL 中而正在嘗試某幾套 NoSQL，現在就來看看 Riak 的使用吧

## 基本環境說明

> 在 Windows 上使用 docker 建立 linux 版本 Riak，接著使用 C# 連線至 Riak 執行基本 CRUD

1. Windows 10 Version 1803 (OS Build 17134.590)
2. Docker Community 18.09.2
3. Bogus 25.0.4
4. Riak 2.1.4
5. RiakClient 2.5.0

    > RiakClient 目前還未推出 .NETStandard 版本

## 建立 Riak instance

Riak 有 Riak KV 與 Riak TS 兩種

- `Riak KV` ：為一般 NoSQL database
- `Riak TS` ：則是為基於 Riak KV 核心功能，針對 Time Series 資料優化的產品

> 透過 docker 僅建立單一 local Riakdb KV instance

```cmd
docker run -d -p 8087:8087 -p 8098:8098 basho/riak-kv
```

  >- 8087 是 Protocol Buffers 使用的
  >- 8098 則是 HTTP 使用

## 使用方式

1. 透過 <http://127.0.0.1:8098/riak/status> 來確認是否有正確安裝及啟動

    ![1startup](https://user-images.githubusercontent.com/3851540/53295022-ec8eaa80-382c-11e9-86fb-db8172834335.png)

2. 設定 C# 連線 config

    ```xml
    <configuration>
    <configSections>
        <section name="riakConfig" type="RiakClient.Config.RiakClusterConfiguration, RiakClient" />
    </configSections>
    <riakConfig nodePollTime="5000" defaultRetryWaitTime="200" defaultRetryCount="3">
        <nodes>
        <node name="dev1" hostAddress="localhost" poolSize="20" />
        </nodes>
    </riakConfig>
    </configuration>
    ```

3. 安裝 `RiakClient` NuGet 套件
    > 本次測試使用 RiakClient  版本為 `2.5.0`

    - Package Manager

        ```bash
        Install-Package RiakClient
        ```

    - .NET CLI

        ```bash
        dotnet add package RiakClient
        ```

4. 實際存取 Riak

    - Insert

        ```cs
        //準備 insert 用假資料
        var _user = GetFakeUserData();
        
        //從 config 中讀取資料
        var cluster = RiakCluster.FromConfig("riakConfig");
        //建立連線
        var client = cluster.CreateClient();
        
        //準備 RiakObject ：指定 UserId 為 key 且 bucket 為 users
        var o = new RiakObject("users", _user.UserId.ToString(), _user);
        //將資料寫入 riak
        client.Put(o);
        ```

    - Select

        ```cs
        //從 config 中讀取資料
        var cluster = RiakCluster.FromConfig("riakConfig");
        //建立連線
        var client = cluster.CreateClient();

        //以 key 來搜尋 users bucket 
        var result = client.Get("users", "40bd4101-4bd5-0d39-3f92-a0575a9670f0");
        //如果成功
        if (result.IsSuccess)
        {
            //將內容轉為 User 物件
            result.Value.GetObject<User>();
        }
        ```

    - Update

        ```cs
        User user = new User();
        //以 key 來搜尋 users bucket 
        var result = client.Get("users", "40bd4101-4bd5-0d39-3f92-a0575a9670f0");
        //如果成功
        if (result.IsSuccess)
        {
            //將內容轉為 User 物件
            user = result.Value.GetObject<User>();
            //將名稱改為 "Yowko"
            user.Name = "Yowko";
            
            //準備 RiakObject ：指定 UserId 為 key 且 bucket 為 users
            var o = new RiakObject("users", user.UserId.ToString(), user);
            //更新資料
            var updateResult = client.Put(o);
        }
        ```

    - Delete

        ```cs
        //以 key 刪除 users bucket 資料
        var deleteResult = client.Delete("users", "40bd4101-4bd5-0d39-3f92-a0575a9670f0");
        ```

## 心得

優點：

- Riak 有支援 Protocol Buffers 猜測在傳輸的效率上應該不差

缺點：

1. 在連線設定上，一定要透過 configSections 來設定，使用上便利性稍差;
2. API 只能透過 key 來搜尋及刪除，會大幅限縮使用情境，
3. 目前仍無 .NETStandard 版本，對於 .Net Core 及跨平台需求的使用者直接就出局了

## 參考資訊

1. [RIAK KV](http://basho.com/products/riak-kv)
2. [basho/riak-dotnet-client](https://github.com/basho/riak-dotnet-client)
3. [Getting Started: CRUD Operations with C Sharp](https://docs.basho.com/riak/kv/2.1.4/developing/getting-started/csharp/crud-operations/)
