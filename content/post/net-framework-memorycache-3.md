---
title: "使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 3 隱藏的效能瓶頸"
date: 2017-02-16T01:42:34+08:00
lastmod: 2018-09-12T00:42:34+08:00
draft: false
tags: ["C#","Cache"]
slug: "net-framework-memorycache-3"
aliases:
    - /2017/02/net-framework-memorycache-cache-part-3.html
    - /2017/02/net-framework-memorycache-3
---
# 使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 3 隱藏的效能瓶頸
之前筆記 [使用 .NET Framework 內建的 MemoryCache 來 Cache 常用資料 - Part 2 使用 lock 避免 ddos db](https://blog.yowko.com/2017/01/net-framework-memorycache-avoid-ddos-db.html) 解決程式可能 ddos db 的重大缺失，最近重新 review code 時發現一個效能瓶頸：取資料時會 lock 所有 MemoryCache 物件而造成所有 access cache 都被 blocking。


## 問題發生原因
1. 測試方法有缺陷
    - 測試方法

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
    - 並非直接測試取資料屬性，而是測試更新資料的方法
2. lock 根物件
    - 取資料方法

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
    - lock 了整個 MemoryCache 物件
## 如何改善
1. 重現問題
    - 修改測試方法

        ```cs 
        Task.Run(() => CacheHelper.Table2Data.Dump());
        Task.Run(() => CacheHelper.TableData.Dump());
        ```
    - 多執行緒仍出現依序執行的現象
        
        ![1error](https://cloud.githubusercontent.com/assets/3851540/23027454/38094412-f49f-11e6-9e11-808263e39921.png)

2. 僅 lock 需要的物件
    - 依 key 來產生唯一個的 lock 物件
        
        ```cs 
        // lock 以 key 產生的專屬 lock object，如果 object 過期或不存在時自動 new 出新的
        // 如果直接 lock cache[key] 會造成無法寫入 cache 資料
        lock (GetCacheObject(key))
        ```
    - 如果 MemoryCache[key] 尚未被建立時需先建立

        ```cs 
        //設定一個 key 用來識別唯一的 lock
        const string LockKey = "@#$%";
        static Object GetCacheObject(string key)
        {
        // 取得每個 Key 的鎖定 object
            string _lockKey =$"{LockKey}{key}";
        //仍然會 lock 整個 memorycahce object 但少了取資料過程  lock 時間會縮短
        lock (_cache)
        {
            // lock object 不存在時直接建立
            if (_cache[_lockKey] == null)
            _cache.Add(_lockKey,new object(),new CacheItemPolicy(){SlidingExpiration = new TimeSpan(0, 10, 0)});
        }
        return _cache[_lockKey];
        }
        ```
    - 因為有先建立暫存物件的關係，更新判斷流程也需調整

        ```cs 
        if ((_cache.Get("TableData") as  List<Table>) == null )
        {
            RefreshTableData();
        }
        ```
    - 調整後完整程式碼

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
                List<Table> listAgency= new List<UserQuery.Table>();
                for (int i = 0; i < 3; i++)
                {
                    listAgency.Add(new Table { Id = Guid.NewGuid(), Name = $"{Guid.NewGuid()}"});
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
                string _lockKey =$"{LockKey}{key}";
                //仍然會 lock 整個 memorycahce object 但少了取資料過程  lock 時間會縮短
                lock (_cache)
                {
                    // lock object 不存在時直接建立
                    if (_cache[_lockKey] == null)
                        _cache.Add(_lockKey,new object(),new CacheItemPolicy(){SlidingExpiration = new TimeSpan(0, 10, 0)});
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
                        if ((_cache.Get("TableData") as  List<Table>) == null )
                        {
                            RefreshTableData();
                        }
                    }
                    return _cache.Get("TableData") as List<Table>;
                }
            }
        }
        ```
## 測試結果
- 兩個 thread 同時啟動 而不是像之前的測試出現等待前一個 thread 執行結束才啟動的現象
    
    ![2result](https://cloud.githubusercontent.com/assets/3851540/23027455/3818abdc-f49f-11e6-8363-1673e5e848b7.png)

- 測試及使用的完整程式碼(可以使用 linqpad 直接執行)
    
    ```cs
    void Main()
    {
        // //先清除 cache 中資料
        MemoryCache _cache = MemoryCache.Default;
        _cache.Remove("TableData");
        _cache.Remove("Table2Data");

        Task.Run(() => CacheHelper.Table2Data.Dump());
        Task.Run(() => CacheHelper.TableData.Dump());

        Console.WriteLine("Done");
    }
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
            List<Table> listAgency= new List<UserQuery.Table>();
            for (int i = 0; i < 3; i++)
            {
                listAgency.Add(new Table { Id = Guid.NewGuid(), Name = $"{Guid.NewGuid()}"});
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
            string _lockKey =$"{LockKey}{key}";
            //仍然會 lock 整個 memorycahce object 但少了取資料過程  lock 時間會縮短
            lock (_cache)
            {
                // lock object 不存在時直接建立
                if (_cache[_lockKey] == null)
                    _cache.Add(_lockKey,new object(),new CacheItemPolicy(){SlidingExpiration = new TimeSpan(0, 10, 0)});
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
                    if ((_cache.Get("TableData") as  List<Table>) == null )
                    {
                        RefreshTableData();
                    }
                }
                return _cache.Get("TableData") as List<Table>;
            }
        }
        public static List<Table> Table2Data
        {
            get
            {
                lock (GetCacheObject("Table2Data"))
                {
                    if ((_cache.Get("Table2Data") as  List<Table>) == null )
                    {
                        RefreshTable2Data();
                    }
                }
                return _cache.Get("Table2Data") as List<Table>;
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
            List<Table> listAgency = new List<UserQuery.Table>();
            for (int i = 0; i < 3; i++)
            {
                listAgency.Add(new Table { Id = Guid.NewGuid(), Name = $"{Guid.NewGuid()}" });
            }

            //設定 cache 過期時間
            CacheItemPolicy cacheItemPolicy = new CacheItemPolicy() { AbsoluteExpiration = DateTime.Now.AddDays(1) };
            //加入 cache
            _cache.Add("Table2Data", listAgency, cacheItemPolicy);
            Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} Stop Job,Now:{DateTime.Now}");
            Console.WriteLine("OK");
        }
    }

    public class Table
    {
        public Guid Id{ get; set; }
        public string Name { get; set; }
    }
    ```

## 心得
又再次體會到寫筆記的好處，如果有個想法立馬紀錄一下，過程中可能還可以順道解決以前自己埋下的地雷XD

# 參考資料
1. [改良式GetCachableData可快取查詢函式](http://blog.darkthread.net/post-2016-04-12-improved-getcachabledata.aspx)