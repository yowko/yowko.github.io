---
title: "RedLock.net 使用 StackExchange.Redis 的連線設定"
date: 2021-10-25T00:39:29+08:00
lastmod: 2021-10-25T00:39:29+08:00
draft: false
tags: ["redis","csharp","dotnet core"]
slug: "redlock-stackexchange-redis-connection"
---

## RedLock.net 使用 StackExchange.Redis 的連線設定

最近專案發現在大量並行處理時會出現重複建立資料的狀況，因為這個專案本來就是設計來進行分散式作業，同時會建立好幾個 instance 在不同的 pod 中，所以預計使用分散式鎖: RedLock.net 來解決問題，關於 RedLock.net 的基本用法在可以參考之前筆記 [使用 RedLock.net 搭配 redis 達成分散式 Lock](/redlocknet-redis-lock/)

團隊中不少專案為了加快資料顯示的速度都使用 redis 做為 cache，其中 redis 的存取都是使用 StackExchange.Redis，而這與 RedLock.net 相依一致，為了統一連線管理為了避免建立多餘的連線，所以打算共用原有的 StackExchange.Redis multiplexer connection，簡單紀錄一下用法

ASP.NET Core 註冊 StackExchange.Redis 的做法可以參考之前筆記 [ASP.NET Core 中 StackExchange.Redis 的註冊與使用方式](/stackexchange-redis-in-aspdotnet-core)

## 基本環境設定

1. mac Big Sur 11.6
2. .NET Core SDK 5.0.401
3. NuGet packages

    - StackExchange.Redis.Extensions.AspNetCore 7.1.1
    - StackExchange.Redis.Extensions.System.Text.Json 7.1.1
    - StackExchange.Redis 2.2.79
    - RedLock.net 2.3.1

## 設定與使用方式

1. 使用 IConnectionMultiplexer 註冊 RedLockFactory

    - `Startup.cs` 中的 `ConfigureServices`

        ```cs
        services.AddSingleton<IConnectionMultiplexer>(ConnectionMultiplexer.Connect(Configuration.GetValue<string>("RedisConnection")));
        
        services.AddSingleton<IDistributedLockFactory,RedLockFactory>(sp =>
            {
                var redis = sp.GetService<IConnectionMultiplexer>();
                var multiplexers = new List<RedLockMultiplexer>
                {
                    (ConnectionMultiplexer)redis.GetDatabase().Multiplexer
                };
                return RedLockFactory.Create( multiplexers);
            });
        ```

    - 使用方式

        ```cs
        public TestController : Controller
        {
            private readonly IConnectionMultiplexer _connectionMultiplexer;
            private readonly IDistributedLockFactory _redlockFactory;

            public TestController(IConnectionMultiplexer multiplexer,IDistributedLockFactory redlockFactory)
            {
                _connectionMultiplexer = multiplexer;
                _redlockFactory = redlockFactory;

                const string resource = "yowko-test-redlock";
                var expiry = TimeSpan.FromSeconds(30);

                using var redLock =  _redlockFactory.CreateLockAsync(resource, expiry).ConfigureAwait(false).GetAwaiter().GetResult();

                if (redLock.IsAcquired)
                {
                    Thread.Sleep(TimeSpan.FromSeconds(10));
                }
            }
        }
        ```

2. 使用 IDatabase 註冊 RedLockFactory

    - `Startup.cs` 中的 `ConfigureServices`

        ```cs
        services.AddSingleton<IDatabase>(ConnectionMultiplexer.Connect(Configuration.GetValue<string>("RedisConnection")).GetDatabase(0));

        services.AddSingleton<IDistributedLockFactory,RedLockFactory>(sp =>
            {
                var redisDb = sp.GetService<IDatabase>();
                var multiplexers = new List<RedLockMultiplexer>
                {
                    (ConnectionMultiplexer)redisDb.Multiplexer
                };
                return RedLockFactory.Create( multiplexers);
            });
        ```

    - 使用方式

        ```cs
        public TestController : Controller
        {
            private readonly IDatabase _redisDb
            private readonly IDistributedLockFactory _redlockFactory;

            public TestController(IDatabase redisDb, IDistributedLockFactory redlockFactory)
            {
                _redisDb = redisDb;
                _redlockFactory = redlockFactory;

                const string resource = "yowko-test-redlock";
                var expiry = TimeSpan.FromSeconds(30);

                using var redLock =  _redlockFactory.CreateLockAsync(resource, expiry).ConfigureAwait(false).GetAwaiter().GetResult();

                if (redLock.IsAcquired)
                {
                    Thread.Sleep(TimeSpan.FromSeconds(10));
                }
            }
        }
        ```

3. 使用擴充方法註冊

    - `Startup.cs` 中的 `ConfigureServices`

        ```cs
        services.AddStackExchangeRedisExtensions<SystemTextJsonSerializer>(Configuration.GetSection("RedisConnection").Get<RedisConfiguration>());
            
        services.AddSingleton<IDistributedLockFactory,RedLockFactory>(sp =>
            {
                var redis = sp.GetService<IRedisCacheClient>();
                var multiplexers = new List<RedLockMultiplexer>
                {
                    (ConnectionMultiplexer)redis.GetDbFromConfiguration().Database.Multiplexer
                };
                return RedLockFactory.Create( multiplexers);
            });
        ```

    - 使用方式

        ```cs
        public TestController : Controller
        {
            private readonly IRedisCacheClient _redisCacheClient;
            private readonly IDistributedLockFactory _redlockFactory;

            public TestController(IRedisCacheClient redisCacheClient,IDistributedLockFactory redlockFactory)
            {
                _redisCacheClient = redisCacheClient;
                _redlockFactory = redlockFactory;

                const string resource = "yowko-test-redlock";
                var expiry = TimeSpan.FromSeconds(30);

                using var redLock =  _redlockFactory.CreateLockAsync(resource, expiry).ConfigureAwait(false).GetAwaiter().GetResult();

                if (redLock.IsAcquired)
                {
                    Thread.Sleep(TimeSpan.FromSeconds(10));
                }
            }
        }
        ```

## 心得

- 實際效果

    ![1redlock](https://user-images.githubusercontent.com/3851540/138641751-48ffd341-a9e6-4377-b37d-64d340322e55.png)

當然實際應用上可以將 RedlockFactory 的建立移至各個實際應用位置，但如果不是對整個程式結構很有把握，可能會有重複 create 的疑慮，所以還是統一註冊管理比較好確保 singleton

## 參考資訊

1. [使用 RedLock.net 搭配 redis 達成分散式 Lock](/redlocknet-redis-lock/)
2. [ASP.NET Core 中 StackExchange.Redis 的註冊與使用方式](/stackexchange-redis-in-aspdotnet-core)
3. [samcook/RedLock.net](https://github.com/samcook/RedLock.net)
4. [RedLockFactory as a Singleton](https://github.com/samcook/RedLock.net/issues/54#issuecomment-504080700)
