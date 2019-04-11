---
title: "使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 4 使用泛型來簡化"
date: 2017-02-17T01:42:34+08:00
lastmod: 2018-09-12T00:42:34+08:00
draft: false
tags: ["C#","Cache"]
slug: "net-framework-memorycache-4"
aliases:
    - /2017/02/net-framework-memorycache-use-generic.html
---
# 使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 4 使用泛型來簡化
經過第一篇文章 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 1 極簡做法](https://blog.yowko.com/2017/01/net-framework-memorycache-simple.html) 介紹了最簡單達到 cache 資料的方法，也在 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 2 使用 lock 避免 ddos db](https://blog.yowko.com/2017/01/net-framework-memorycache-avoid-ddos-db.html) 先解決程式可能 ddos db 的重大缺失, 加上無意中發現的 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 3 隱藏的效能瓶頸](https://blog.yowko.com/2017/02/net-framework-memorycache-cache-part-3.html),接著就是來改善程式未使用泛型而造成程式碼雜亂的問題. 

## 前情提要
- CacheHelper
    
    ```cs
    public class CacheHelper
    {
    	private static MemoryCache _cache = MemoryCache.Default;
    	//設定一個 key 用來識別唯一的 lock
    	const string LockKey = "@#$%";
    	/// <summary>
    	/// 更新 TableData
    	/// </summary>
    	public static void RefreshTableData()
    	{
    		Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} Start Job,Now:{DateTime.Now}");
    		Thread.Sleep(3000);
    
    		//移除 cache 中資料
    		_cache.Remove("TableData");
    
    		//存取資料
    		List<Table> listAgency = new List<UserQuery.Table>();
    		for (int i = 0; i < 3; i++)
    		{
    			listAgency.Add(new Table { Id = Guid.NewGuid(), Name = $"{Guid.NewGuid()}" });
    		}
    
    		//設定 cache 過期時間
    		CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() { AbsoluteExpiration = DateTime.Now.AddDays(1) };
    		//加入 cache
    		_cache.Add("TableData", listAgency, cacheItemPolicy);
    		Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} Stop Job,Now:{DateTime.Now}");
    		Console.WriteLine("OK");
    
    	}
    	/// <summary>
    	/// 依 key 取得 cache object
    	/// </summary>
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
    	///  TableData
    	/// </summary>
    	public static List<Table> TableData
    	{
    		get
    		{
    			//加上 lock
    			lock (GetCacheObject("TableData"))
    			{
    				if ((_cache.Get("TableData") as List<Table>) == null)
    				{
    					RefreshTableData();
    				}
    			}
    			return _cache.Get("TableData") as List<Table>;
    		}
    	}
	}
    ```

## 修改開始
1. CacheHelper
	- 刪掉 property 及 原本取資料方法
	- 加上新的泛型方法
        
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
        ```
    - 完整範例
        
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

2. 實際使用
	- 使用新方法
		`CacheHelper.GetCacheData<{modelType}>("{cacheKey}",{refresh=false});`
	- `List<Table>` 是 caceh 存放型別
	- 第一個參數 `cacheKey` 是用來區隔多組 cache data 的 key (如需多個相同 table 不同 cache ，可藉由 cache 區隔)
	- 第二個參數 `refresh` 是用來強制更新 cache 的

## 心得
透過使用 .NET Framework 內建的 MemoryCache 讓我們可以很方便地 cache 資料也很快速的完成相關程式碼，但你發現了嗎？ 這幾篇 cache 相關文章都是使用 local memory 來達到 cache 目的，如果 server 較多或是 server 啟動時間不一時很容易造成資料不一致的問題，接著我們就來看該如何使用 cache server 


# 參考資料
1. [改良式GetCachableData可快取查詢函式](http://blog.darkthread.net/post-2016-04-12-improved-getcachabledata.aspx)
2. [Dynamic table names in Entity Framework linq](http://stackoverflow.com/questions/31033055/dynamic-table-names-in-entity-framework-linq)



