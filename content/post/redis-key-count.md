---
title: "取得 Redis 中指定 key 條件的筆數"
date: 2018-07-13T02:40:00+08:00
lastmod: 2020-12-11T02:40:52+08:00
draft: false
tags: ["C#","Redis"]
slug: "redis-key-count"
aliases:
    - /2018/07/redis-key-count.html
---
# 取得 Redis 中指定 key 條件的筆數
公司有個流量很大的 ASP.NET MVC 網站仍在使用 Session，並利用 Redis 來儲存 Session 資訊，而近來的大型活動讓 Redis 壓力倍增，使用的 memory 是活動前的 2-3 倍，於是興起清查 Redis 各式來源使用量的念頭

經過嘗試了幾種做法，對最後解決方式的執行時間還算滿意，順手紀錄一下各種方式的優缺點待日後備查

## 使用 `KEYS`
- 官方文件請參考：[KEY](https://redis.io/commands/keys)
- 用法：`KEYS pattern`
- 說明：會列出所有符合 `pattern` 的 key
- 實際範例
    - 語法
        
        ```
        keys urn:session:*
        ``` 
    - 結果 
        
        ![1keys](https://user-images.githubusercontent.com/3851540/42651912-60aceb12-8643-11e8-9aff-fa23aaca0164.png)
- 缺點：Redis Key 數量大時會造成 blocking
    - 超過 101 萬 key 列出所有 key 需耗時 35.87 秒
        
        ![2keys1m](https://user-images.githubusercontent.com/3851540/42651913-60d8fc16-8643-11e8-8af0-1fcbf2675402.png)

## 使用 `SCAN`
- 官方文件請參考：[SCAN](https://redis.io/commands/scan)
- 用法：`SCAN cursor [MATCH pattern] [COUNT count]`
- 說明：
    
    > 透過 cursor 來批次取回符合 `pattern` 指定 `count` 數量的 key，透過重複指定回傳的 cursor 當做參數直到 cursor 為 `0`  即完整取回所有 key
    
    - `MATCH` 用來指定 key 比對條件
    - `COUNT` 預設為 `10`，一次取回 `10` 筆
- 實際範例
    
    > 符合 `urn:session:*` 有 18 筆為例，
    
    - 語法與結果
        1. cursor 為 `0` 取得預設 10 筆資訊
            
            ```
            scan 0 MATCH urn:session:*
            ``` 
            
            ![3scan1](https://user-images.githubusercontent.com/3851540/42651914-610123c6-8643-11e8-838b-b9a4a65e2426.png)
        2. cursor 為 `2` 得到 cursor 重置為 `0` 取完所有內容
            
            ```
            scan 2 MATCH urn:session:*
            ```
            
            ![4scan2](https://user-images.githubusercontent.com/3851540/42651915-612726fc-8643-11e8-827f-395ade4fd208.png)
- 缺點：數量大時逐一輸入不僅耗時且容易出錯

## 使用 C# 的 StackExchange.Redis 套件

> 前面有提到 `KEYS` 會造成 blocking 而 `SCAN` 則是操作不便容易出錯，所以就該使用更簡單的做法來達成目的：`StackExchange.Redis 的 Keys` 語法，需要特別注意的是 `Keys` 是針對 `server` 跟平常儲存資料的針對 database 語法不同，另外 StackExchange.Redis 沒有為 `SCAN` 建立專屬的語法，執行 `Keys` 時只要能轉換為 `SCAN` 的行為皆會使用 `SCAN` 指令，無法轉換的才會使用 `KEYS` 指令
    
1. 寫法一：將 `server` 加至 redis factory 中
    
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
2. 寫法二：程式中指定 server
    - redis factory
        
        ```cs
        public static class RedisConnectionFactory
        {
            private static readonly Lazy<ConnectionMultiplexer> Connection;
            static RedisConnectionFactory()
            {
                #region -- sample 1 --
                var connectionString = "127.0.0.1:6379,127.0.0.1:6380,password=password";
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
            }
            public static ConnectionMultiplexer GetConnection => Connection.Value;
            public static IDatabase RedisDB => GetConnection.GetDatabase();
        }
        ```
    - 指定 server
        
        ```cs
        //取得第一個的 Redis EndPoint
        var item = RedisConnectionFactory.GetConnection.GetEndPoints(true).FirstOrDefault();
        //透過 Redis EndPoint 取得 Redis server
        var server = RedisConnectionFactory.GetConnection.GetServer(item);
        ``` 
3. 實際取得 keys 及筆數
    - 取得所有 key
        
        ```cs
        server.Keys();
        ``` 
    - 指定 pattern 取得所有 key
        
        ```cs
        server.Keys(pattern:"urn:session:*",cursor:0);
        ``` 
    - 指定 pattern 的 key 數量 
        
        ```cs
        server.Keys(pattern:"urn:session:*",cursor:0).Count()
        ``` 
    - key 數量很多時記得加上 `pageSize`
        
        ```cs
        server.Keys(pattern:"urn:session:*",cursor:0,pageSize:10000)
        ``` 
4. 實際效果比較
    
    > 約莫為 1百萬資料 
    
    - 未加 `pageSize`，預設使用 `10`：耗時 `41.4` 秒
        
        ![5keys](https://user-images.githubusercontent.com/3851540/42651918-614ec4e6-8643-11e8-833f-8f3d488a9f0b.png)
    - 加上 `pageSize:10000`：耗時 `4.5` 秒
        
        ![6keys](https://user-images.githubusercontent.com/3851540/42651919-61775cd0-8643-11e8-87c1-831ca3948481.png) 

## 心得
原本收到需求時，就傻傻地直接去開 GUI 透過 pattern 過濾筆數，約 30 秒後 GUI crash 了XD，立馬改透過 redis-cli 直接下 `KEYS` (心臟真是大顆 @@") 雖然跑得出結果但 blocking 其他指令超過 30 秒，這多跑個幾次絕對會被吊起來打呀

雖然嘗試使用 `SCAN`，但跑幾個 cursor 後我自己就亂掉了，所幸查到可以用 StackExchange.Redis 的 Keys 指令來處理，加上背後也是走 `SCAN` 對效能影響較小，只是使用預設值 (COUNT:10) 執行時間上並沒有改善，經過嘗試不同的參數後執行時間才有明顯的下降

雖然影響時間縮短許多，但 StackExchange.Redis 仍建議不該在 production 環境上使用 `Keys` 指令，若真的有需要也建議在 slave 上執行以縮小影響範圍

# 參考資訊
1. [KEY](https://redis.io/commands/keys)
2. [SCAN](https://redis.io/commands/scan)
3. [如何使用 StackExchange.Redis 取得所有 keys 值與指定 pattern 的 key](/2017/04/stackexchange-redis-get-all-keys-or-pattern.html)
4. [Where are KEYS, SCAN, FLUSHDB etc?](https://stackexchange.github.io/StackExchange.Redis/KeysScan)