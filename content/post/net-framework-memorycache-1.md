---
title: "使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 1 極簡做法"
date: 2017-01-30T00:42:34+08:00
lastmod: 2018-09-10T00:42:34+08:00
draft: false
tags: ["C#","Cache"]
slug: "net-framework-memorycache-1"
aliases:
    - /2017/01/net-framework-memorycache-simple.html
---
# 使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 1 極簡做法
程式多多少少有些資料或設定是經常需要使用的，如果這些資料異動頻率低的特性就可以考慮將其加入 cache，以節省每次使用都要 access 資料庫或是 call 外部 api 所造成的成本，也能讓程式執行效能獲得提升。

首先就來看看不借助外部工具，單純使用 .NET Framework 內建特性可以如何達成吧

##  加入 dll 參考

> MemoryCache 雖然是 .NET Framework 內建特性，但預設未被加至 GAC 中，需要手動加入參考

1. 專案視窗 Reference 右鍵 --> Add Reference...
	
    ![1reference](https://cloud.githubusercontent.com/assets/3851540/22261920/be0d7892-e2a9-11e6-99e2-6257d5fd3107.png)
2. 在 Assemblies tab 下，搜尋 `System.Runtime.Caching`
	- 注意：要使用 4.0.0.0 版本(有些環境可能有 2.0.0.0 版本)
		
        ![2addref](https://cloud.githubusercontent.com/assets/3851540/22261921/be14df06-e2a9-11e6-8ff7-e89fe84b1ba7.png)

## 加入 CacheHelper.cs

> 用來負責處理需要 cache 的資料內容

1. 將 class 改為 static
	
    ```cs
    public static class CacheHelper
    ```

2. 引用 `System.Runtime.Caching`
	
    ```cs
    using System.Runtime.Caching;
    ```
3. 定義 MemoryCache 的私有欄位 (field)
	
    ```cs
    private static MemoryCache _cache = MemoryCache.Default;
    ```
4. 定義所需資料屬性 (property)
	- 僅需 getter
        
        ```cs
        /// <summary>
        ///  TableData
        /// </summary>
        public static List<Table> TableData
        {
            get
            {
                if (!_cache.Contains("TableData"))
                    RefreshTableData();
                return _cache.Get("TableData") as List<Table>;
            }
        }
        ```
5. 加上更新方法
	- 取資料時只取常使用的欄位以節省資源
 
        ```cs
        /// <summary>
        /// 更新 TableData
        /// </summary>
        public static void RefreshTableData()
        {
            //移除 cache 中資料
            _cache.Remove("TableData");

            //存取資料
            TestEnumEntities db = new TestEnumEntities();
            var listAgency = db.Table.ToList();

            //設定 cache 過期時間
            CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() {AbsoluteExpiration = DateTime.Now.AddDays(1) };
            //加入 cache
            _cache.Add("TableData", listAgency, cacheItemPolicy);
        }
        ```

## 實際使用
- 引用 CacheHelper 的 namespace 後直接使用
	
    ```cs
    var tableData=CacheHelper.TableData;
    ```

## 心得與缺陷
我一直以來都是使用上面介紹的方法來實現簡易 Cache ，使用上相當便利，經歷幾個較大專案考驗後發現問題，接著又看到黑大的文章 [改良式GetCachableData可快取查詢函式](http://blog.darkthread.net/post-2016-04-12-improved-getcachabledata.aspx) 驚覺有重大缺陷, 為瞭解決缺點，會透過後面的文章來逐步改善上面程式碼的問題。

1. 沒有加上 lock 機制
	
    > 可能造成程式會去大量去 DB 取資料 ，形成 DDOS

2. 沒有使用泛型
	
    > 需要加入新資料到 Cache 時，會有重複的 code 出現

# 參考資料
1. [MemoryCache 方法](https://msdn.microsoft.com/zh-tw/library/system.runtime.caching.memorycache_methods.aspx)
2. [改良式GetCachableData可快取查詢函式](http://blog.darkthread.net/post-2016-04-12-improved-getcachabledata.aspx)