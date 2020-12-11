---
title: "如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 1 (使用 Strings 型別)"
date: 2017-02-18T01:42:34+08:00
lastmod: 2020-12-11T00:42:34+08:00
draft: false
tags: ["C#","Cache","Redis"]
slug: "dotnet-redis-strings"
aliases:
    - /2017/02/use-redis-cache-server-for-dotnet.html
    - /2017/02/dotnet-redis-strings
---
# 如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 1 (使用 Strings 型別)
在前面文章 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 1 極簡做法](/2017/01/net-framework-memorycache-simple.html) 介紹了最簡單達到 cache 資料的方法，也[使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 2 使用 lock 避免 ddos db](/2017/01/net-framework-memorycache-avoid-ddos-db.html)，加上意外發現的[使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 3 隱藏的效能瓶頸](/2017/02/net-framework-memorycache-cache-part-3.html)以及最後在 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 4 使用泛型來簡化](/2017/02/net-framework-memorycache-use-generic.html) 中，將 cache 的 function 做了一些簡化，但也想到了新的問題：目前的 cache 都是使用 application server 的 local memory，多台 server 的情境還是可能對 db 產生壓力加上還可能因為 cache 取得時間的落差而有資料不一致的問題，所以我們接著就來將原本使用 .NET Framework 內建 MemoryCache 的 cahce 改使用 Redis 做為共用的 Cache Server


## 原本程式碼

```cs
public static class CacheHelper
{
	static MemoryCache _cache = MemoryCache.Default;
	//設定一個 key 用來識別唯一的 lock
	const string LockKey = "@#$%";
	static Object GetCacheObject(string key)
	{
		// 取得每個 Key 的鎖定 object
		string _lockKey = $"{LockKey}{key}";
		//仍然會 lock 整個 memorycahce object 但少了取資料過程  lock 時間會縮短
		lock (_cache)
		{
			// lock object 不存在時直接建立
			if (_cache[_lockKey] == null)
				_cache.Add(_lockKey, new object(), new CacheItemPolicy() { SlidingExpiration = new TimeSpan(0, 10, 0) });
		}
		return _cache[_lockKey];
	}
	/// <summary>
	/// 取得 cache data
	/// </summary>
	/// <typeparam name="T">Model 型別</typeparam>
	/// <param name="key">caceh key</param>
	/// <param name = "refresh" >是否強制更新 cache </param >
	/// <returns>回傳 Model 的 list</returns>
	public static List<T> GetCacheData<T>(string key, bool refresh = false) where T : class
	{
		var aa = _cache[key];
		// lock 以 key 產生的專屬 lock object，如果 object 過期會自動 new 出新的
		// 如果直接 lock cache[key] 會造成無法寫入 cache 資料
		lock (GetCacheObject(key))
		{
			// 取得 memorycache 是否已有 key 的 cahce 資料
			List<T> cacheData = _cache[key] as List<T>;
			//是否強制更新 cache
			if (cacheData != null && refresh)
			{
				_cache.Remove(key);
				cacheData = null;
			}
			//cache 不存在
			if (cacheData == null)
			{
				//存取資料
				TestEnumEntities db = new TestEnumEntities();
				//將傳入的 model 型別至 db 取得資料後轉為 list
				cacheData = db.Set(typeof(T)).ToListAsync().Result.OfType<T>().ToList();
				//設定 cache 過期時間
				CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() { SlidingExpiration = new TimeSpan(0, 10, 0) };
				//加入 cache
				_cache.Add(key, cacheData, cacheItemPolicy);
			}
			return cacheData;
		}
	}
}
```

## 建立 Redis 連線
1. 使用 StackExchange.Redis 來存取 Redis
    
    > `Install-Package StackExchange.Redis`
2. 管理 `ConnectionMultiplexer`
    - 建立使用 `Singleton` 模式
    - 具有完全的 thread-safe
    - 設計來共享及重用的
    - 不該每次存取皆建立
3. 建立 `RedisConnectionFactory`
    
    ```cs
    public static class RedisConnectionFactory
    {
        private static readonly Lazy<ConnectionMultiplexer> Connection;
        static RedisConnectionFactory()
        {
            var connectionString =
                System.Configuration.ConfigurationManager.AppSettings["RedisConnection"];
            var options = ConfigurationOptions.Parse(connectionString);
            Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
        }
        public static ConnectionMultiplexer GetConnection() => Connection.Value;
    }
    ```

## web.config 加上連線字串

```xml
<appSettings>
    <add key="RedisConnection" value="localhost:6379"/>
</appSettings>
```

## 將原本使用 `MemoryCache` 修改為使用 Redis
1. 加入 `ConnectionMultiplexer` 宣告
    
    >private static ConnectionMultiplexer _redisconnection; 
2. 加入 `IDatabase` 宣告
    
    > private static IDatabase _db;
3. 建構式初始化 `ConnectionMultiplexer` 與 `IDatabase`
    
    ```cs
    static RedisLockCacheHelper()
    {
        _redisconnection = RedisConnectionFactory.GetConnection();
        _db = _redisconnection.GetDatabase();
    }
    ```
4. 修改取 cahce data 方法
    
    ```cs
    /// <summary>
    /// 取得 cache data
    /// </summary>
    /// <typeparam name="T">Model 型別</typeparam>
    /// <param name="key">caceh key</param>
    /// <param name = "refresh" >是否強制更新 cache </param >
    /// <returns>回傳 Model 的 list</returns>
    public static List<T> GetCacheData<T>(string key, bool refresh = false) where T : class
    {
        // 取得 memorycache 是否已有 key 的 cahce 資料
        List<T> cacheData = _db.KeyExists(key) ? JsonConvert.DeserializeObject<List<T>>(_db.StringGet(key)) : null;
        //是否強制更新 cache
        if (cacheData != null && refresh)
        {
            _db.KeyDelete(key);
            cacheData = null;
        }
        //cache 不存在
        if (cacheData == null)
            cacheData = SetCacheData<T>(key);
        return cacheData;

    }
    ```
5. 一樣加入 lock (寫入 redis 一筆資料，寫完再移除：我只想到這個笨方法，請指導我一下XD)
    
    ```cs
    /// <summary>
    /// 設定 data cache
    /// </summary>
    /// <typeparam name="T">Model 型別</typeparam>
    /// <param name="key">caceh key</param>
    /// <returns></returns>
    private static List<T> SetCacheData<T>(string key)
    {
        List<T> cacheData = new List<T>();
        string _lockKey = $"{LockKey}{key}";
        if (_db.StringGet(_lockKey).IsNull && !_db.KeyExists(key))
        {
            //標示為寫入中
            _db.StringSet(_lockKey, true.ToString());
            //存取資料
            TestEnumEntities db = new TestEnumEntities();
            //將傳入的 model 型別至 db 取得資料後轉為 list
            cacheData = db.Set(typeof(T)).ToListAsync().Result.OfType<T>().ToList();
            //設定 cache 過期時間
            TimeSpan cacheItemPolicy = new TimeSpan(1, 0, 0, 0);
            //加入 cache
            _db.StringSet(key, JsonConvert.SerializeObject(cacheData), cacheItemPolicy);
            //移除寫入中標示
            _db.KeyDelete(_lockKey);
        }
        else
        {
            if (!_db.KeyExists(key))
                cacheData = SetCacheData<T>(key);
            else
                cacheData = JsonConvert.DeserializeObject<List<T>>(_db.StringGet(key));
        }
        return cacheData;
    }
    ```
6. 完整程式
    
    ```cs
    public static class RedisLockCacheHelper
    {
        private static ConnectionMultiplexer _redisconnection;
        private static IDatabase _db;
        //設定一個 key 用來識別唯一的 lock
        const string LockKey = "@#$%";
        static RedisLockCacheHelper()
        {
            _redisconnection = RedisConnectionFactory.GetConnection();
            _db = _redisconnection.GetDatabase();
        }

        /// <summary>
        /// 取得 cache data
        /// </summary>
        /// <typeparam name="T">Model 型別</typeparam>
        /// <param name="key">caceh key</param>
        /// <param name = "refresh" >是否強制更新 cache </param >
        /// <returns>回傳 Model 的 list</returns>
        public static List<T> GetCacheData<T>(string key, bool refresh = false) where T : class
        {
            // 取得 memorycache 是否已有 key 的 cahce 資料
            List<T> cacheData = _db.KeyExists(key) ? JsonConvert.DeserializeObject<List<T>>(_db.StringGet(key)) : null;
            //是否強制更新 cache
            if (cacheData != null && refresh)
            {
                _db.KeyDelete(key);
                cacheData = null;
            }
            //cache 不存在
            if (cacheData == null)
                cacheData = SetCacheData<T>(key);
            return cacheData;

        }
        /// <summary>
        /// 設定 data cache
        /// </summary>
        /// <typeparam name="T">Model 型別</typeparam>
        /// <param name="key">caceh key</param>
        /// <returns></returns>
        private static List<T> SetCacheData<T>(string key)
        {
            List<T> cacheData = new List<T>();
            string _lockKey = $"{LockKey}{key}";
            if (_db.StringGet(_lockKey).IsNull && !_db.KeyExists(key))
            {
                //標示為寫入中
                _db.StringSet(_lockKey, true.ToString());
                //存取資料
                TestEnumEntities db = new TestEnumEntities();
                //將傳入的 model 型別至 db 取得資料後轉為 list
                cacheData = db.Set(typeof(T)).ToListAsync().Result.OfType<T>().ToList();
                //設定 cache 過期時間
                TimeSpan cacheItemPolicy = new TimeSpan(1, 0, 0, 0);
                //加入 cache
                _db.StringSet(key, JsonConvert.SerializeObject(cacheData), cacheItemPolicy);
                //移除寫入中標示
                _db.KeyDelete(_lockKey);
            }
            else
            {
                if (!_db.KeyExists(key))
                    cacheData = SetCacheData<T>(key);
                else
                    cacheData = JsonConvert.DeserializeObject<List<T>>(_db.StringGet(key));
            }
            return cacheData;
        }
    }
    ```

## 實際使用
- 使用新方法
		
    ```cs
    RedisLockCacheHelper.GetCacheData<{modelType}>("{cacheKey}",{refresh=false});
    ```
	
    - `List<Table>` 是 caceh 存放型別
	- 第一個參數 `cacheKey` 是用來區隔多組 cache data 的 key (如需多個相同 table 不同 cache ，可藉由 cache 區隔)
	- 第二個參數 `refresh` 是用來強制更新 cache 的

## 結果
使用 [Redis Desktop Manager](https://redisdesktop.com/) 來確認結果

![result](https://cloud.githubusercontent.com/assets/3851540/22986889/f2df01b8-f3e7-11e6-95dd-59653394f4d2.png)

## 心得
經過前面幾篇文章的基礎，要轉換成使用 Redis Cache server，是不是沒有想像中困難？！修改過後我們初步有了集中式的 cache 機制了。目前我們將 object 轉為 string 來儲存，接著我們就來利用 Redis 支援的其他型別來更有效率地利用 Redis cache

# 參考資料
1. [Using Redis Cache with .NET and C#](http://www.codefluff.com/using-redis-cache-with-net-and-c/)
2. [Redis Desktop Manager](https://redisdesktop.com/)
3. [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 1 極簡做法](http://blog.yowko.com/2017/01/net-framework-memorycache-simple.html)
4. [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 2 使用 lock 避免 ddos db](http://blog.yowko.com/2017/01/net-framework-memorycache-avoid-ddos-db.html)
5. [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 3 隱藏的效能瓶頸](http://blog.yowko.com/2017/02/net-framework-memorycache-cache-part-3.html)
6. [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 4 使用泛型來簡化](http://blog.yowko.com/2017/02/net-framework-memorycache-use-generic.html)
