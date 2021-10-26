---
title: "如何使用 StackExchange.Redis 配合 Sentinel 或是 Cluster 達到高可用性"
date: 2017-03-25T02:44:34+08:00
lastmod: 2021-10-15T00:42:34+08:00
draft: false
tags: ["Redis","套件","csharp"]
slug: "stackexchange-redis-sentinel-cluster"
aliases:
    - /2017/03/stackexchangeredis-sentinel-cluster.html
---
## 如何使用 StackExchange.Redis 配合 Sentinel 或是 Cluster 達到高可用性

前陣子公司其他 team 的同事遇到了 Redis 的 production issue，詳細情形我不好多問，大意是懷疑 Redis 異常，造成部份服務無法正常運作，所以針對 Redis 相關設定進行檢討，就發現了幾個問題：

1. Master-Slave 沒有正確設定
2. 沒有設定 Sentinel 的 failover 機制
3. 程式的連線字串也是寫死固定連線至 master

之前文章 [Windows 環境如何設定 Redis Master-Slave 與 Sentinel](//blog.yowko.com/2017/03/windows-redis-master-slave-sentinel.html) 已經介紹了該如何在設定 Master-Slave 環境以及建立 Sentinel faiover機制，Redis server 算是已經做到初步 HA 了，接著當然就是程式面也要能配合，下面內容就是介紹如何在 StackExchange.Redis 設定 Sentinel 或是 Cluster

## 為什麼需要程式配合

我們來看在之前文章 [如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 2 (使用 Hashes 型別)](//blog.yowko.com/2017/02/dotnet-use-redis-hash.html) 或是 Redis 相關系列文章也行，其中的 `RedisConnectionFactory` 部份

```CS
private static readonly Lazy<ConnectionMultiplexer> Connection;
    static RedisConnectionFactory()
    {
    var connectionString = "localhost:6379";
    var options = ConfigurationOptions.Parse(connectionString);
    Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
    }
 public static ConnectionMultiplexer GetConnection => Connection.Value;
 public static IDatabase RedisDB => GetConnection.GetDatabase();
```

這段程式碼中，使用的 connectionString 是固定寫死在程式中的("localhost:6379")。而就算這個 connectionString 改設定在 config 檔案中，還是一樣無法解決問題：因為程式本身並不知道 Sentinel 將哪個 slave 提昇為 master，也不知道什麼時候會執行，這就是為什麼 Redis HA 需要程式面配合的原因。

## 如何調整程式

我們可以利用 StackExchange.Redis ConnectionMultiplexer `自動辨識 master` 的特性來做為調整依據，詳細內容請參考 [StackExchange.Redis Basic Usage](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md)

![1automaster](https://cloud.githubusercontent.com/assets/3851540/24083525/573ea7be-0d13-11e7-9f71-c0ec5a0a7d88.png)

我們可以同時給多組 Redis instance 連線資訊，讓 StackExchange.Redis ConnectionMultiplexer 自行判斷應該使用哪個連線，以下內容會延續使用上面的 `RedisConnectionFactory` 當做範例來進行修改，寫法有兩個可以自由選擇適合情境

1. 程式中使用 ConfigurationOptions object 指定 EndPoints

    ```cs
    ConfigurationOptions options = new ConfigurationOptions
    {
        EndPoints ={{ "127.0.0.1", 6379 },{ "127.0.0.1", 6380 }}
    };
    ```

    - 完整範例

        ```cs
        public static class RedisConnectionFactory
        {
            private static readonly Lazy<ConnectionMultiplexer> Connection;
            static RedisConnectionFactory()
            {
                ConfigurationOptions options = new ConfigurationOptions
                {
                    EndPoints ={{ "127.0.0.1", 6379 },{ "127.0.0.1", 6380 }}
                };
                Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
            }
            public static ConnectionMultiplexer GetConnection => Connection.Value;
            public static IDatabase RedisDB => GetConnection.GetDatabase();
        }
        ```

2. 使用連線字串

    ```cs
    var connectionString = "127.0.0.1:6379,127.0.0.1:6380";
    ```

    - 完整範例

        ```cs
        public static class RedisConnectionFactory
        {
            private static readonly Lazy<ConnectionMultiplexer> Connection;
            static RedisConnectionFactory()
            {
                var connectionString = "127.0.0.1:6379,127.0.0.1:6380";
                var options = ConfigurationOptions.Parse(connectionString);
                Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
            }
            public static ConnectionMultiplexer GetConnection => Connection.Value;
            public static IDatabase RedisDB => GetConnection.GetDatabase();
        }
        ```

## 結果驗證

> 情境說明：
>
> - master :6379
> - slave  :6380
> - 皆使用預設設定
> - sentinel:26379

1. 情境 1：針對 Master(6379) 寫入資料，確認有同步
    - Master -Slave 機制正確運作
    - `var connectionString = "127.0.0.1:6379";`
    - `RedisConnectionFactory.RedisDB.StringSet("情境 1","針對 6379 寫入資料，確認有同步");`
    - 結果：成功寫入 6379 並成功同步 6380

        ![2s1](https://cloud.githubusercontent.com/assets/3851540/24083526/5764381c-0d13-11e7-9a98-bdc17fa13414.png)

2. 情境 2：針對 Slave(6380) 寫入資料，確認無法寫入
    - Slave (6380) 預設唯讀
    - `var connectionString = "127.0.0.1:6380";`
    - `RedisConnectionFactory.RedisDB.StringSet("情境 2","針對 6380 寫入資料，確認無法寫入");`
    - 結果：無法寫入

        ![3fail](https://cloud.githubusercontent.com/assets/3851540/24083527/577e41b2-0d13-11e7-9048-bcbd181f6382.png)

3. 情境 3：shutdown Master(6379)，並使用 `var connectionString = "127.0.0.1:6379,127.0.0.1:6380";` 寫入，確認資料是否寫入
    - Sentinel 有正常發揮作用，對 Redis 做 failover
    - `var connectionString = "127.0.0.1:6379,127.0.0.1:6380";`
    - `RedisConnectionFactory.RedisDB.StringSet("情境 3","shutdown 6379，並使用 127.0.0.1 6379 及127.0.0.1 6380 寫入，確認資料是否寫入");`
    - 結果：正確寫入 6380

        ![4slaveok](https://cloud.githubusercontent.com/assets/3851540/24083528/57845f3e-0d13-11e7-87a7-545a5e0f4437.png)

4. 情境 4：啟動 `原 Master(6379)`，確認資料是否同步
    - 原 master 回復後仍可正確提供服務 - 會自動變為 slave
    - 結果：正確同步至 `原 Master(6379)`

        ![5syncok](https://cloud.githubusercontent.com/assets/3851540/24083529/5785af42-0d13-11e7-9989-b86cf5a04b53.png) 

## 參考資料

1. [StackExchange.Redis Basic Usage](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md)
2. [Auto discover cluster nodes](https://github.com/StackExchange/StackExchange.Redis/issues/300)
