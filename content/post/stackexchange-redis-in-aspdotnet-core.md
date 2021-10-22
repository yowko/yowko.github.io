---
title: "ASP.NET Core 中 StackExchange.Redis 的註冊與使用方式"
date: 2021-10-22T00:39:29+08:00
lastmod: 2021-10-22T00:39:29+08:00
draft: false
tags: ["redis","csharp","dotnet core","ASP.NET Core"]
slug: "stackexchange-redis-in-aspdotnet-core"
---

## ASP.NET Core 註冊 StackExchange.Redis 的方式

最近有個專案需要用到 RedLock.net，在做可行性評估時發現 StackExchange.Redis 有幾個註冊方式自己都沒有紀錄過，趁著工作空檔簡單筆記一下

## 基本環境說明

1. mac Big Sur 11.6
2. .NET Core SDK 5.0.401
3. docker desktop 3.6.0(67351)
4. docker images
    - redis 6.2.6
5. NuGet packages

    - StackExchange.Redis.Extensions.AspNetCore 7.1.1
    - StackExchange.Redis.Extensions.System.Text.Json 7.1.1
    - StackExchange.Redis 2.2.79

6. redis cluster

    > 因為團隊使用 redis cluster，今天就以 redis cluster 做範例，建立方式可以參考之前筆記 [使用 Docker Compose 建立 Redis Cluster](/docker-compose-redis-cluster/) 或是直接下載 docker-compose [yowko/docker-compose-redis-cluster](https://github.com/yowko/docker-compose-redis-cluster)

## 註冊方式

- 使用 singleton 方式註冊 IConnectionMultiplexer

    1. 安裝 NuGet 套件：`StackExchange.Redis`
    2. config 新增 redis 連線字串

        ```json
        "RedisConnection": "192.168.80.3:7000,192.168.80.3:7001,192.168.80.3:7002,192.168.80.3:7003,192.168.80.3:7004,192.168.80.3:7005, password=pass.123"
        ```

    3. 註冊 `IConnectionMultiplexer`

        > `Startup.cs` --> `ConfigureServices` 方法

        ```cs
        services.AddSingleton<IConnectionMultiplexer>(ConnectionMultiplexer.Connect(Configuration.GetValue<string>("RedisConnection")));
        ```

    4. 使用方式

        > 使用 DI 取得 redis IConnectionMultiplexer，並在  key 不存在時寫入 `key`:yowko 與 `value`:test 至 `db0` 並指定 expire 為 `60 秒`

        ```cs
        public TestController : Controller
        {
            private readonly IConnectionMultiplexer _connectionMultiplexer;

            public TestController(IConnectionMultiplexer multiplexer)
            {
                _connectionMultiplexer = multiplexer;
                _connectionMultiplexer.GetDatabase(0).StringSetAsync("yowko", "test", TimeSpan.FromSeconds(60), When.NotExists);
            }
        }
        ```

- 使用 singleton 方式註冊 IDatabase

    > 因為 redis cluster 只有 `db0` 對於 redis cluster 使用者沒有切換 database 需求，直接註冊 IDatabase 對於後續使用比較方便

    1. 安裝 NuGet 套件：`StackExchange.Redis`
    2. config 新增 redis 連線字串

        ```json
        "RedisConnection": "192.168.80.3:7000,192.168.80.3:7001,192.168.80.3:7002,192.168.80.3:7003,192.168.80.3:7004,192.168.80.3:7005, password=pass.123"
        ```

    3. 註冊 `IDatabase`

        > `Startup.cs` --> `ConfigureServices` 方法

        ```cs
        services.AddSingleton<IDatabase>(ConnectionMultiplexer.Connect(Configuration.GetValue<string>("RedisConnection")).GetDatabase(0));
        ```

    4. 使用方式

        > 使用 DI 取得 redis IDatabase，並在 key 不存在時寫入 `key`:yowko 與 `value`:test 至 `db0` 並指定 expire 為 `60 秒`

        ```cs
        public TestController : Controller
        {
            private readonly IDatabase _redisDb;

            public TestController(IDatabase redisDb)
            {
                _redisDb = redisDb;
                _redisDb.StringSetAsync("yowko", "test", TimeSpan.FromSeconds(60), When.NotExists);
            }
        }
        ```

- 使用擴充方法註冊

    1. 安裝 NuGet 套件：`StackExchange.Redis.Extensions.AspNetCore` 與 `StackExchange.Redis.Extensions.System.Text.Json`
    2. config 新增 redis 連線字串

        ```json
        "RedisConnection": {
          "Password": "pass.123",
          "Hosts": [
            {
              "Host": "192.168.80.3",
              "Port": "7000"
            },
            {
              "Host": "192.168.80.3",
              "Port": "7001"
            },
            {
              "Host": "192.168.80.3",
              "Port": "7002"
            },
            {
              "Host": "192.168.80.3",
              "Port": "7003"
            },
            {
              "Host": "192.168.80.3",
              "Port": "7004"
            },
            {
              "Host": "192.168.80.3",
              "Port": "7005"
            }
          ]
        }
        ```

    3. 註冊 `IConnectionMultiplexer`

        > `Startup.cs` --> `ConfigureServices` 方法

        ```cs
        services.AddStackExchangeRedisExtensions<SystemTextJsonSerializer>(Configuration.GetSection("RedisConnection").Get<RedisConfiguration>());
        ```

    4. 使用方式

        > 使用 DI 取得 redis IRedisCacheClient，並在 key 不存在時寫入 `key`:yowko 與 `value`:test 至 `db0` 並指定 expire 為 `60 秒`

        ```cs
        public TestController : Controller
        {
            private readonly IRedisCacheClient _redisCacheClient;

            public TestController(IRedisCacheClient redisCacheClient)
            {
                _redisCacheClient = redisCacheClient;
                _redisCacheClient.Db0.AddAsync("yowko","test", TimeSpan.FromSeconds(60), When.NotExists);
            }
        }
        ```

## 心得

原本覺得直接將 `IConnectionMultiplexer` 或是 `IDatabase` 註冊為 singleton 有點醜，感覺用 extention method 註冊比較帥，不過 IRedisCacheClient 的缺點讓我無法忽視：

1. IRedisCacheClient 的屬性直接寫死 db0-db16，用到不存在的 db 會拋 exception

    ![1listalldb](https://user-images.githubusercontent.com/3851540/138432289-2dbf8e5e-1c72-4d48-b3e1-0d3e1fe32e2b.png)

2. config 的設定較繁瑣

    ![2config](https://user-images.githubusercontent.com/3851540/138435926-72430dfc-85fa-49d7-b4ba-a254019aa72c.png)

仔細想想我還是用老方法好了XD

## 參考資訊

1. [使用 Docker Compose 建立 Redis Cluster](/docker-compose-redis-cluster/)
2. [yowko/docker-compose-redis-cluster](https://github.com/yowko/docker-compose-redis-cluster)
3. [Using StackExchange.Redis in a ASP.NET Core Controller](https://newbedev.com/using-stackexchange-redis-in-a-asp-net-core-controller)
4. [USING REDIS CACHE WITH ASP.NET CORE 3.1 USING STACKEXCHANGE.REDIS.EXTENSIONS.CORE EXTENSIONS](https://tutexchange.com/using-redis-cache-with-asp-net-core-3-1-using-stackexchange-redis-extensions-core-extensions/)
