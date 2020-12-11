---
title: "使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 5 使用動態條件與動態欄位"
date: 2017-03-11T01:42:34+08:00
lastmod: 2020-12-11T00:42:34+08:00
draft: false
tags: ["C#","Cache"]
slug: "net-framework-memorycache-5"
aliases:
    - /2017/03/dotnet-expression-predicatebuilder.html
---
# 使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 5 使用動態條件與動態欄位
一直以為 使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 4 使用泛型來簡化 是 .NET MemoryCache 系列的最後一篇，壓根忘記還有個重要功能還沒寫：使用動態條件來取資料及針對動態欄位 cache

使用動態條件來取資料：前面文章使用泛型來簡化資料取得，但當時共用的資料取得流程中並沒有加上任何的條件判斷，所以會不經任何條件過濾取得所有資料
針對動態欄位 cache：跟上面一樣的問題，共用的流程中會將 table 中所有欄位皆 cache，如果 table 不需 cache 的欄位很多，無形中就造成資源上的浪費

## 前情提要

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

## 修改開始
1. 取資料方法加上傳入與傳出的型別
    
    >GetCacheData<TEntry, TResult>

2. 取資料方法加入使用動態條件的參數

    >Expression<Func<TEntry, bool>> predicate

3. 取資料方法加入使用動態欄位的參數
    
    >Expression<Func<TEntry, TResult>> selector

4. 原本取資料的實際行為加上使用動態條件與動態欄位
    
    ```cs
    //將傳入的 model 型別至 db 取得資料後轉為 list
    cacheData = db.Set(typeof(TEntry))
        .ToListAsync()
        .Result.OfType<TEntry>()
        .AsQueryable()
        .Where(predicate)
        .Select(selector)
        .ToList();
    ```
5. 完整範例

     ```cs
     /// <summary>
     /// 取得 cache data
     /// </summary>
     /// <typeparam name="TEntry">傳入 model 型別</typeparam>
     /// <typeparam name="TResult">輸出 model 型別</typeparam>
     /// <param name="predicate">動態條件</param>
     /// <param name="key">cache key</param>
     /// <param name="selector">動態欄位 selector</param>
     /// <param name="refresh">是否強制更新 cache</param>
     /// <returns>回傳 List&lt;TResult&gt;</returns>
     public static List<TResult> GetCacheData<TEntry, TResult>(Expression<Func<TEntry, bool>> predicate, string key, Expression<Func<TEntry, TResult>> selector, bool refresh = false) where TEntry : class where TResult : class
     {
         var aa = _cache[key];
         // lock 以 key 產生的專屬 lock object，如果 object 過期會自動 new 出新的
         // 如果直接 lock cache[key] 會造成無法寫入 cache 資料
         lock (GetCacheObject(key))
         {
             // 取得 memorycache 是否已有 key 的 cahce 資料
             List<TResult> cacheData = _cache[key] as List<TResult>;
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
                 SNManagerEntities db = new SNManagerEntities();
                 //將傳入的 model 型別至 db 取得資料後轉為 list
                 cacheData = db.Set(typeof(TEntry))
                                .ToListAsync()
                                .Result.OfType<TEntry>()
                                .AsQueryable()
                                .Where(predicate)
                                .Select(selector)
                                .ToList();
                 //設定 cache 過期時間
                 CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() { SlidingExpiration = new TimeSpan(0, 10, 0) };
                 //加入 cache
                 _cache.Add(key, cacheData, cacheItemPolicy);
             }
             return cacheData;
         }
     }
     ```
6. 實際使用
    - 指定動態條件
        
        ```cs
        //建立動態條件
        var predicate = PredicateBuilder.True<{modelType}>();
        predicate = predicate.And(a => a.IsEnable == true);
        predicate = predicate.And(a => a.IsDelete == false);
        ```
    - 指定動態欄位

        ```cs 
        //建立動態欄位 selector
        Expression<Func<{modelType}, dynamic>> selector = a => new { a.APIKey };
        ```
    - 使用新方法 
        
        ```cs
        CacheService.GetCacheData<{modelType}, dynamic>(predicate, "{cacheKey}", selector,{refresh=false}); CacheHelper.GetCacheData<{modelType}>("{cacheKey}",{refresh=false});
        ```
    - `{modelType}` 是 caceh 存放型別
    - 第一個參數 `cacheKey` 是用來區隔多組 cache data 的 key (如需多個相同 table 不同 cache ，可藉由 cache 區隔)
    - 第二個參數 `refresh` 是用來強制更新 cache 的
    - LINQPad 可以直接執行, .NET 請參考這篇 [Dynamically Composing Expression Predicates](http://www.albahari.com/nutshell/predicatebuilder.aspx) 安裝

# 參考資訊
1. [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 4 使用泛型來簡化](/2017/02/net-framework-memorycache-use-generic.html)
2. [dynamically specify select columns linq](http://stackoverflow.com/questions/38467681/dynamically-specify-select-columns-linq)
3. [Dynamic Column Name in LinQ](http://stackoverflow.com/questions/24732724/dynamic-column-name-in-linq)
4. [CONSTRUCT DYNAMIC FILTERS WITH C#](http://www.pashov.net/code/dynamic+filters)
5. [Dynamically Composing Expression Predicates](http://www.albahari.com/nutshell/predicatebuilder.aspx)