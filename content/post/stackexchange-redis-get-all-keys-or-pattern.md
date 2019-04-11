---
title: "如何使用 StackExchange.Redis 取得所有 keys 值與指定 pattern 的 key"
date: 2017-04-27T01:00:00+08:00
lastmod: 2018-09-18T01:00:00+08:00
draft: false
tags: ["Redis","套件"]
slug: "stackexchange-redis-get-all-keys-or-pattern"
aliases:
    - /2017/04/stackexchange-redis-get-all-keys-or-pattern.html
---
# 如何使用 StackExchange.Redis 取得所有 keys 值與指定 pattern 的 key
同事因專案需要打算將 redis 資料與 db 資料進行比對，為了要比對資料，首先就是將 redis 資料導出，所以需要取得所有 keys，需求初聽覺得應該是滿容易的，語法就是 `redis-cli keys *` 就會取得所有 key 了，但透過 StackExchange.Redis 遇到了些障礙，筆記一下

## 前提條件

*   連線管理
    
    ```cs
    public static class RedisConnectionFactory
    {
        private static readonly Lazy<ConnectionMultiplexer> Connection;
        static RedisConnectionFactory()
        {
            #region -- sample 1 --
            var connectionString = "127.0.0.1:6379,127.0.0.1:6380,assword=password";
            var options = ConfigurationOptions.Parse(connectionString);
            #endregion
            #region -- sample 2 --
            //  var options = new ConfigurationOptions()
            //  {
            //   EndPoints = { { "127.0.0.1",6379 }, { "127.0.0.1", 6380}},
            //   Password="password"
            //  };
            #endregion
            Connection = new Lazy<ConnectionMultiplexer>(() => nnectionMultiplexer.Connect(options));
        }
        public static ConnectionMultiplexer GetConnection => Connection.Value;
        public static IDatabase RedisDB => GetConnection.GetDatabase();
    }
    ```

## 如何取得所有 key 及指定 pattern

  >依 StackExchange.Redis 的設計，想要拿到 redis 所有 keys 需要針對 redis server 執行指令，而基礎連線管理原本並不會建立 server 的物件，因此我們得先加上 redis server 的定義

*   加上 redis server

    ```cs
    public static class RedisConnectionFactory
    {
        private static readonly Lazy<ConnectionMultiplexer> Connection;
        public static IServer RedisServer;
        static RedisConnectionFactory()
        {
            #region -- sample 1 --
            var connectionString = "localhost:6379,password=password";
            var options = ConfigurationOptions.Parse(connectionString);
            #endregion
            #region -- sample 2 --
            //  var options = new ConfigurationOptions()
            //  {
            //   EndPoints = { { "127.0.0.1",6379 }, { "127.0.0.1", 6380}},
            //   Password="password"
            //  };
            #endregion
            Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
            RedisServer = GetConnection.GetServer(options.EndPoints.First());
        }
        public static ConnectionMultiplexer GetConnection => Connection.Value;
        public static IDatabase RedisDB => GetConnection.GetDatabase();
    }
    ```

*   取得所有 key

    > `RedisConnectionFactory.RedisServer.Keys();`

*   取得特定 pattern

    > `RedisConnectionFactory.RedisServer.Keys(pattern: "{pattern}");`

**如果執行速度緩慢請參 [取得 Redis 中指定 key 條件的筆數](https://blog.yowko.com/2018/07/redis-key-count.html)**

# 參考資訊
1.  [Get all keys from Redis Cache database](http://stackoverflow.com/questions/37436429/get-all-keys-from-redis-cache-database)
2.  [How can I get a key count for StackExchange.Redis?](http://stackoverflow.com/questions/41514786/how-can-i-get-a-key-count-for-stackexchange-redis)
