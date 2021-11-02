---
title: "使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 2 使用 lock 避免 ddos db"
date: 2017-01-31T00:42:34+08:00
lastmod: 2021-08-18T00:42:34+08:00
draft: false
tags: ["csharp","Cache"]
slug: "net-framework-memorycache-2"
aliases:
    - /2017/01/net-framework-memorycache-avoid-ddos-db.html
---
## 使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 2 使用 lock 避免 ddos db

經過前一篇文章 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 1 極簡做法](/2017/01/net-framework-memorycache-simple.html) 介紹了最簡單達到 cache 資料的方法，文末也提到數個已知的問題，首先優先來處理程式可能 ddos db 的重大缺失

## 前情提要

- CacheHelper

    ```cs
    public static class CacheHelper
    {
        private static MemoryCache _cache = MemoryCache.Default;
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

        /// <summary>
        /// 更新 TableData
        /// </summary>
        public static void RefreshTableData()
        {
            //移除 cache 中資料
            _cache.Remove("TableData");

            //存取資料
            TestEnumEntities db = new TestEnumEntities();
            var listAgency = db.Table.Select(a=>a.WeekDay).ToList();

            //設定 cache 過期時間
            CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() {AbsoluteExpiration = DateTime.Now.AddDays(1) };
            //加入 cache
            _cache.Add("TableData", listAgency, cacheItemPolicy);
        }
    }
    ```

## 測試程式

修改自黑大文章 [改良式GetCachableData可快取查詢函式](http://blog.darkthread.net/post-2016-04-12-improved-getcachabledata.aspx)

- 程式進入點

    ```cs
    void Main()
    {
        //先清除 cache 中資料
        MemoryCache _cache = MemoryCache.Default;
        _cache.Remove("TableData");
        //起三個 thread
        var tasks = new List<Task>();
        for (var i = 0; i < 3; i++)
        {
            tasks.Add(Task.Factory.StartNew(() =>
            {
                var data = CacheHelper.TableData;
                Console.WriteLine("Data:" + JsonConvert.SerializeObject(data));
            }));
        }
        tasks.ForEach(t => t.Wait());
        Console.WriteLine("Done");
    }
    ```

    ![1runtriple](https://cloud.githubusercontent.com/assets/3851540/22261955/e5b39de0-e2a9-11e6-8dc5-46dbbaf9a555.png)

- 更新 cache

    > 我把執行 sleep 跟 輸出訊息的部份搬到執行更新 cache 的方法中以便於測試

    ```cs
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
        TestEnumEntities db = new TestEnumEntities();
        var listAgency = db.Table.ToList();

        //設定 cache 過期時間
        CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() { AbsoluteExpiration = DateTime.Now.AddDays(1) };
        //加入 cache
        _cache.Add("TableData", listAgency, cacheItemPolicy);
        Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} Stop Job,Now:{DateTime.Now}");
        Console.WriteLine("OK");

    }
    ```

## 加入 lock

```cs
/// <summary>
///  TableData
/// </summary>
public static List<Table> TableData
{
    get
    {
        MemoryCache _cache = MemoryCache.Default;
        //加上 lock
        lock (_cache)
        {
            if (!_cache.Contains("TableData"))
            {
                RefreshTableData();
            }
        }
        return _cache.Get("TableData") as List<Table>;
    }
}
```

![2runonce](https://cloud.githubusercontent.com/assets/3851540/22261956/e5b501e4-e2a9-11e6-913e-5bfb279c40d6.png)

## 其他測試

- 多個 proprty 下會不會造成 block ？

    >不會

- 測試流程
    1. 加入新的 property 及更新方法至 cachehelper

        ```cs
        public static List<Table2> Table2Data
        {
            get
            {

                lock (_cache)
                {
                    if (!_cache.Contains("Table2Data"))
                    {
                        RefreshTable2Data();
                    }
                }
                return _cache.Get("Table2Data") as List<Table2>;
            }
        }
        /// <summary>
        /// 更新 TableData
        /// </summary>
        public static void RefreshTable2Data()
        {
            Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} Start Job,Now:{DateTime.Now}");
            Thread.Sleep(10000);

            //移除 cache 中資料
            _cache.Remove("Table2Data");

            //存取資料
            TestEnumEntities db = new TestEnumEntities();
            var listAgency = db.Table2.Select(a => a.TestStr).ToList();

            //設定 cache 過期時間
            CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() { AbsoluteExpiration = DateTime.Now.AddDays(1) };
            //加入 cache
            _cache.Add("Table2Data", listAgency, cacheItemPolicy);
            Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} Stop Job,Now:{DateTime.Now}");
            Console.WriteLine("OK");
        }
        ```

    2. 將兩個 property sleep 時間設一長一短
        - RefreshTable2Data sleep(10000)
        - RefreshTableData sleep(3000)

    3. 先起一個 thread 執行 sleep 時間較長的 property 更新方法，再起另一個 thread 執行 sleep 時間較短的 property 更新方法

        ```cs
        void Main()
        {
            MemoryCache _cache = MemoryCache.Default;
            _cache.Remove("TableData");
            _cache.Remove("Table2Data");

            // sleep 時間較久先跑
            Task.Run(() => CacheHelper.RefreshTable2Data());
            Task.Run( () => CacheHelper.RefreshTableData());

            Console.WriteLine("Done");
        }
        ```

  - 測試結果

    - sleep 時間短的還是先完成，沒有被另一個 thread block

        ![3mutipleproperty](https://cloud.githubusercontent.com/assets/3851540/22261954/e5b1f4cc-e2a9-11e6-9375-455b82b282c8.png)

## 心得

修改的工非常少(只在 getter 中加上 lock)，倒是花在建立測試情境的時間比較長，雖然程式碼不優，還是解決了問題，後續就來修改成使用泛型的版本以配合多個 property

## 參考資料

1. [MemoryCache 方法](https://msdn.microsoft.com/zh-tw/library/system.runtime.caching.memorycache_methods.aspx)
2. [改良式GetCachableData可快取查詢函式](http://blog.darkthread.net/post-2016-04-12-improved-getcachabledata.aspx)
