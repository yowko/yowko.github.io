---
title: "使用 RedLock.net 搭配 redis 達成分散式 Lock"
date: 2018-04-10T01:31:00+08:00
lastmod: 2018-10-04T01:31:44+08:00
draft: false
tags: ["套件","C#","Redis"]
slug: "redlocknet-redis-lock"
aliases:
    - /2018/04/redlocknet-redis-lock.html
---
# 使用 RedLock.net 搭配 redis 達成分散式 Lock
RedLock.net 是前兩個禮拜從安德魯大大的 [架構面試題 #1, 線上交易的正確性](http://columns.chicken-house.net/2018/03/25/interview01-transaction/) 一文中看到使用 redis 搭配 RedLock 演算法製造出分散式鎖定 (Distributed locks) 的套件，也是 Redlock distributed lock algorithm 在 C# 上的實作之一，主要相依於 StackExchange.Redis 套件(實作 Redlock 的其他程式語言及套件可以參考 [Distributed locks with Redis](https://redis.io/topics/distlock))

當下看到安德魯大大介紹，立馬回想起過去為了達到分散式鎖定苦思了許多但還是沒有想到好方法的冏況，終於有機會突破當時的技術瓶頸，恰巧最近需要重構一段程式碼從本機的 object lock 改為分散式鎖定 (Distributed locks)，正好可以透過實戰來上手效果更佳

## 如何使用 RedLock.net
1. 系統啟動時：使用 Redis 連線資訊建立 RedLockFactory
    - RedLock 自行管理連線
        
        ```cs
        var endPoints = new List<RedLockEndPoint>
        {
        	new DnsEndPoint("redis1", 6379),
        	new DnsEndPoint("redis2", 6379),
        	new DnsEndPoint("redis3", 6379)
        };
        var redlockFactory = RedLockFactory.Create(endPoints);
        ```
    - 共用 StackExchange.Redis 連線
        
        ```cs
        var existingConnectionMultiplexer1 = ConnectionMultiplexer.Connect("redis1:6379");
        var existingConnectionMultiplexer2 = ConnectionMultiplexer.Connect("redis2:6379");
        var existingConnectionMultiplexer3 = ConnectionMultiplexer.Connect("redis3:6379");
        
        var multiplexers = new List<RedLockMultiplexer>
        {
        	existingConnectionMultiplexer1,
        	existingConnectionMultiplexer2,
        	existingConnectionMultiplexer3
        };
        var redlockFactory = RedLockFactory.Create(multiplexers);
        ```
2. 執行 lock
    - lock or give up (取得 resource lock 權就做事，否則就放棄)
        
        ```cs
        var resource = "lock_key";//lock object
        var expiry = TimeSpan.FromSeconds(30);//lock object 失效時間
        
        using (var redLock = await redlockFactory.CreateLockAsync(resource, expiry)) // 有非 async 的版本
        {
        	// 確定取得 lock 所有權
        	if (redLock.IsAcquired)
        	{
        		// 執行需要獨佔資源的核心工作
        	}
        }
        // 脫離 using 範圍自動就會解除 lock
        ```
    - lock, retry or wait to give up(取得 resource lock 權就做事，未取得就等指定 retry 時間後重試至指定 wait 時間後放棄)
        
        ```cs
        var resource = "lock_key";//lock object
        var expiry = TimeSpan.FromSeconds(30);//lock object 失效時間
        var wait = TimeSpan.FromSeconds(10);//放棄重試時間
        var retry = TimeSpan.FromSeconds(1);//重試間隔時間
        
        // blocks 直到取得 lock 資源或是達到放棄重試時間
        using (var redLock = await redlockFactory.CreateLockAsync(resource, expiry, wait, retry)) // 有非 async 的版本
        {
        	// 確定取得 lock 所有權
        	if (redLock.IsAcquired)
        	{
        		// 執行需要獨佔資源的核心工作
        	}
        }
        // 脫離 using 範圍自動就會解除 lock
        ```
3.  系統關閉時：Dispose RedLockFactory
    
    ```cs
    redlockFactory.Dispose();
    ```

## 情境說明

> 有個 web api 有可能在瞬間收到多個重複的 request，為了避免造成值在重複 request 同時處理下造成異常，同事使用 object lock 來解決

- 模擬程式碼
    
    > 原始程式碼中沒有那麼多 log 資訊，為了方便釐清狀況多加一些 log
    
    ```cs
    static class helper
    {
    	private static object _lock = new object();
    	public static void CheckReceiveBet(string membercode)
    	{
    		
    		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: received.");
    		lock (_lock)
    		{
    			Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock start. at {DateTime.Now}");
    			Thread.Sleep(5*1000);
    			Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock end.at {DateTime.Now}");
    		}
    
    		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: done.");
    	}
    }
    ```
## 可能的問題
1. 該 api 被部署至多台機器上
    
    > application 的 object lock 只限於單台機器上

2. lock object 沒有加入其他參數概念
    
    > 會造成 block 所有 request

## 實際狀況
- 使用 Task 模擬多個 request：lock 導致 single thread 處理 request
    
    ```cs
    Task.Run(() =>
	{
		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}: start.");
		helper.CheckReceiveBet("123");
	});
	Task.Run(() =>
	{
		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}: start.");
		helper.CheckReceiveBet("ABC");
	});
    ```
    
    >![1objectlock](https://user-images.githubusercontent.com/3851540/38511000-44cb0246-3c59-11e8-919a-d1d8dfb8da3c.png)

- 需要在 lock object 加入 request 資訊，避免無差別 block 所有 request
    
    ```cs
    static class helper
    {
    	private static object _lock = new object();
    	public static void CheckReceiveBet(string membercode)
    	{
    
    		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: received.");
    		lock (_lock + membercode)
    		{
    			Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock start. at {DateTime.Now}");
    			Thread.Sleep(5 * 1000);
    			Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock end.at {DateTime.Now}");
    		}
    
    		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: done.");
    	}
    }
    ```
    
    >![2objectwithmember](https://user-images.githubusercontent.com/3851540/38511002-450f6d28-3c59-11e8-8921-1e1f53d6b44f.png)

## 使用 RedLock.net 達成分散式 Lock
透過加入 request (以本例來說將 `membercode` 加至 lock 物件) 相關內容即可避免 block 所有 request，但並沒有解決多個 instance 同時處理的狀況，立馬來看看該如何實現分散式 lock

1. 建立 RedisConnectionFactory 與 RedLockFactory
    
    ```cs
    public static class RedisConnectionFactory
    {
        private static readonly Lazy<ConnectionMultiplexer> Connection;
        private static readonly RedLockFactory _redlockFactory;
        static RedisConnectionFactory()
        {
            var connectionString = "127.0.0.1:6379";
            var options = ConfigurationOptions.Parse(connectionString);
            Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
        }
        public static ConnectionMultiplexer GetConnection() => Connection.Value;
        public static RedLockFactory RedisLockFactory
        {
            get
            {
                var multiplexers = new List<RedLockMultiplexer> { RedisConnectionFactory.GetConnection() };
                return RedLockFactory.Create(multiplexers);
            }
        }
    }
    ```
2. 將原本的 object lock 換為 RedLock.net
    - lock or give up
        
        > 取得 lock 資源;取得 lock 則放棄執行
        
        ```cs
        static class helper
        {
        	public static void CheckReceiveBet(string membercode)
        	{
        
        		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: received.");
        		var resource = $"lockkey_{membercode}";//resource lock key
		        var expiry = TimeSpan.FromSeconds(30);//lock key expire 時間
        
        		// 傳入 resource lock key 與 expiry
        		using (var redLock = RedisConnectionFactory.RedisLockFactory.CreateLockAsync(resource, expiry).Result)
        		{
        			// 確定取得 lock 所有權
        			if (redLock.IsAcquired)
        			{
        				Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock start. at {DateTime.Now}");
        				Thread.Sleep(5 * 1000);
        				Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock end.at {DateTime.Now}");
        			}
        			else
        				Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}:Not get the locker");
        		}
        		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: done.");
        	}
        }
        ```
        
        >![3lockorgiveup](https://user-images.githubusercontent.com/3851540/38511003-4539776c-3c59-11e8-9746-42b70f7621fb.png)
    
    - lock, retry and wait
        
        > 取得 lock 資源;未取得 lock 之前，並依指定間隔時間 (retry) 重試，直到達到指定放棄時間 (wait)
        
        ```cs
        static class helper
        {
        	public static void CheckReceiveBet(string membercode)
        	{
        
        		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: received.");
        		var resource = $"lockkey_{membercode}";//resource lock key
        		var expiry = TimeSpan.FromSeconds(30);//lock key expire 時間
        		var wait = TimeSpan.FromSeconds(10);//放棄重試時間
        		var retry = TimeSpan.FromSeconds(1);//重試間隔時間
        
        		//傳入 resource lock key , expiry, wait, retry
        		using (var redLock = RedisConnectionFactory.RedisLockFactory.CreateLockAsync(resource, expiry, wait, retry).Result)
        		{
        			// 確定取得 lock 所有權
        			if (redLock.IsAcquired)
        			{
        				Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock start. at {DateTime.Now}");
        				Thread.Sleep(5 * 1000);
        				Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: lock end.at {DateTime.Now}");
        			}
        			else
        				Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}:Not get the locker");
        		}
        		Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{membercode}: done.");
        	}
        }
        ```
        
        >![4lockwaitretry](https://user-images.githubusercontent.com/3851540/38511004-4562d5e4-3c59-11e8-863d-e64903872846.png)

## 心得
印象上前段日子我也嘗試利用 redis 來達成分散式鎖定，只是當時只透過寫入特定 key 至 redis 來檢查是否取得獨佔權，但完全沒有考慮過 cluster node 及資料碰撞問題，幸虧安德魯大大的 [架構面試題 #1, 線上交易的正確性](http://columns.chicken-house.net/2018/03/25/interview01-transaction/) 一文才讓我學到正確方式，感謝安德魯大大


# 參考資訊
1. [架構面試題 #1, 線上交易的正確性](http://columns.chicken-house.net/2018/03/25/interview01-transaction/)
2. [Distributed locks with Redis](https://redis.io/topics/distlock)