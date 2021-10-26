---
title: "StackExchange 連線 Redis 出現 Timeout"
date: 2017-06-29T21:00:00+08:00
lastmod: 2021-10-15T21:00:02+08:00
draft: false
tags: ["套件","Debug","Redis"]
slug: "stackexchange-redis-timeout"
aliases:
    - /2017/06/stackexchange-redis-timeout.html
---
## StackExchange 連線 Redis 出現 Timeout

同事反應出現大量 Redis 連線 Timeout 的錯誤，因為 Redis 上存放 Sessoion 跟許多 config cache 資料，如果 Redis 異常會嚴重影響線上服務，所以立馬需要進行除錯

首先使用 Redis-cli 確認服務仍正常執行中，接著執行了 Redis Benchmark 檢查 server 回應，數據並沒有發現異常，使用 Redis Desktop Manager 連線 Redis 資料也可以正常取得，推測 Redis server 本身應該是正常的

接著確認同事的使用情境後發現並非全面性出現 Redis timeout 只有存取幾個特定的 key 會出現問題。仔細檢查後發現：引起 timeout error 的 key 都有 size 較大的特徵，推測可能是資料量太大造成的

## 錯誤訊息

- 錯誤訊息

    ```txt
    2017-06-28 18:04:56,025 [17] [ERROR] [RedisObjectStore`1] 
    Inner exception number - 0
    Timeout performing HGETALL faqscache:******.portal.bll.appdatamanager+cachedata, inst: 1, queue: 18, qu: 0, qs: 18, qc: 0, wr: 0, wq: 0, in: 0, ar: 0, clientName: TestAPP01, serverEndpoint: 127.0.0.1:8188, keyHashSlot: 3414, IOCP: (Busy=0,Free=1000,Min=2,Max=1000), WORKER: (Busy=6,Free=8185,Min=2,Max=8191) (Please take a look at this article for some common client-side issues that can cause timeouts: http://stackexchange.github.io/StackExchange.Redis/Timeouts)
        at StackExchange.Redis.ConnectionMultiplexer.ExecuteSyncImpl[T](Message message, ResultProcessor`1 processor, ServerEndPoint server)
        at StackExchange.Redis.RedisBase.ExecuteSync[T](Message message, ResultProcessor`1 processor, ServerEndPoint server)
        at StackExchange.Redis.RedisDatabase.HashGetAll(RedisKey key, CommandFlags flags)
        at ******.Portal.BLL.Redis.RedisObjectStore`1.Get(String key)
    Main exception & its inner exception Log end
    ```

## 解決方式

* **修改資料同步 timeout 設定 - 放寬 syncTimeout 時間 (預設 1000 毫秒)**

1. 原始 redis 連線資訊

    ```cs
    public static class RedisConnectionFactory
    {
    
        private static readonly Lazy<ConnectionMultiplexer> Connection;
        public static IServer RedisServer;
        
        static RedisConnectionFactory()
        {
            var connectionString = "127.0.0.1:6379,127.0.0.1:6380,password=password";
            var options = ConfigurationOptions.Parse(connectionString);
            Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
            RedisServer = GetConnection.GetServer(options.EndPoints.First());
        }
        public static ConnectionMultiplexer GetConnection => Connection.Value;
        public static IDatabase RedisDB => GetConnection.GetDatabase();
    }
    ```

2. 修改後

    ```cs
    public static class RedisConnectionFactory
    {
        private static readonly Lazy<ConnectionMultiplexer> Connection;
        public static IServer RedisServer;
        static RedisConnectionFactory()
        {
            var connectionString = "127.0.0.1:6379,127.0.0.1:6380,password=password,syncTimeout =3000";
            var options = ConfigurationOptions.Parse(connectionString);
            Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
            RedisServer = GetConnection.GetServer(options.EndPoints.First());
        }
        public static ConnectionMultiplexer GetConnection => Connection.Value;
        public static IDatabase RedisDB => GetConnection.GetDatabase();
    }
    ```

## 心得

放寬資料同步時間後，timeout 問題確實獲得解決，但還是建議同事從源頭端來縮小 redis 的資料量大小，一來減少 network 傳輸的 io ，二來可以增加 redis 回應速度

## 參考資訊

1. [StackExchange.Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration)
2. [使用StackExchange.Redis客戶端進行Redis訪問出現的Timeout異常排查](http://www.cnblogs.com/JiaK/p/5945554.html)
