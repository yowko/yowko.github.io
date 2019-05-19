---
title: "關於 ASP.NET Core IMemoryCache RegisterPostEvictionCallback 的觸發時機"
date: 2019-05-14T21:30:00+08:00
lastmod: 2019-05-14T21:30:31+08:00
draft: false
tags: ["ASP.NET Core MVC","ASP.NET Core","Cache"]
slug: "aspdotnet-core-imemorycache-registerpostevictioncallback"
---

# 關於 ASP.NET Core IMemoryCache RegisterPostEvictionCallback 的觸發時機

同事提到想用 ASP.NET Core 的 IMemoryCache 來處理 application 本身的 cache，無奈小弟學藝不精沒有太多想法可以參與討論，所以趕緊惡補，藉這個機會學習也順便做些 POC，大致上與過去使用 `System.Runtime.Caching` 的 `MemoryCache` 用法概念接近，但實際用法則不全然相同

## 基本比較

-|MemoryCache(ASP.NET Core)|MemoryCache(.NET Framework)
:---|:---|:---
Namespace|Microsoft.Extensions.Caching.Memory|System.Runtime.Caching
Assembly|Microsoft.Extensions.Caching.Memory.dll|System.Runtime.Caching.dll
繼承|Object --> MemoryCache|Object --> ObjectCache --> MemoryCache
實作|IMemoryCache,IDisposable|IEnumerable,IDisposable
建構子|MemoryCache(IOptions＜MemoryCacheOptions＞)|MemoryCache(String, NameValueCollection)<br/>MemoryCache(String, NameValueCollection, Boolean)
屬性|Count|CacheMemoryLimit<br/>Default<br/>DefaultCacheCapabilities<br/>Item[String]<br/>Name<br/>PhysicalMemoryLimit<br/>PollingInterval
方法|Compact(Double)<br/>CreateEntry(Object)<br/>Dispose()<br/>Dispose(Boolean)<br/>Finalize()<br/>Remove(Object)<br/>TryGetValue(Object, Object)|Add(CacheItem, CacheItemPolicy)<br/>AddOrGetExisting(CacheItem, CacheItemPolicy)<br/>AddOrGetExisting(String, Object, CacheItemPolicy, String)<br/>AddOrGetExisting(String, Object, DateTimeOffset, String)<br/>Contains(String, String)<br/>CreateCacheEntryChangeMonitor(IEnumerable＜String＞, String)<br/>Dispose()<br/>Get(String, String)	<br/>GetCacheItem(String, String)<br/>GetCount(String)<br/>GetEnumerator()	<br/>GetLastSize(String)	<br/>GetValues(IEnumerable<String>, String)<br/>Remove(String, CacheEntryRemovedReason, String)<br/>Remove(String, String)<br/>Set(CacheItem, CacheItemPolicy)<br/>Set(String, Object, CacheItemPolicy, String)<br/>Set(String, Object, DateTimeOffset, String)<br/>Trim(Int32)
明確介面實作 |-|IEnumerable.GetEnumerator()
擴充方法|Get(IMemoryCache, Object)<br/>Get＜TItem＞(IMemoryCache, Object)<br/>GetOrCreate＜TItem＞(IMemoryCache, Object, Func<ICacheEntry,TItem>)<br/>GetOrCreateAsync＜TItem＞(IMemoryCache, Object, Func＜ICacheEntry,Task＜TItem＞＞)<br/>Set＜TItem＞(IMemoryCache, Object, TItem)<br/>Set＜TItem＞(IMemoryCache, Object, TItem, MemoryCacheEntryOptions)<br/>Set＜TItem＞(IMemoryCache, Object, TItem, IChangeToken)<br/>Set＜TItem＞(IMemoryCache, Object, TItem, DateTimeOffset)<br/>Set＜TItem＞(IMemoryCache, Object, TItem, TimeSpan)<br/>TryGetValue＜TItem＞(IMemoryCache, Object, TItem)|CopyToDataTable＜T＞(IEnumerable＜T＞)<br/>CopyToDataTable＜T＞(IEnumerable＜T＞, DataTable, LoadOption)<br/>CopyToDataTable＜T＞(IEnumerable＜T＞, DataTable, LoadOption, FillErrorEventHandler)<br/>Cast＜TResult＞(IEnumerable)<br/>OfType＜TResult＞(IEnumerable)<br/>AsParallel(IEnumerable)<br/>AsQueryable(IEnumerable)<br/>Ancestors＜T＞(IEnumerable＜T＞)<br/>Ancestors＜T＞(IEnumerable＜T＞, XName)<br/>DescendantNodes＜T＞(IEnumerable＜T＞)<br/>Descendants＜T＞(IEnumerable＜T＞)<br/>Descendants＜T＞(IEnumerable＜T＞, XName)<br/>Elements＜T＞(IEnumerable＜T＞)<br/>Elements＜T＞(IEnumerable＜T＞, XName)<br/>InDocumentOrder＜T＞(IEnumerable＜T＞)<br/>Nodes＜T＞(IEnumerable＜T＞)<br/>Remove＜T＞(IEnumerable＜T＞)

## 基本環境說明

1. macOS Mojave 10.14.4
2. .NET Core SDK 2.2.107
3. NuGet package
    - NLog 4.6.3
    - NLog.Web.AspNetCore 4.8.2
4. ASP.NET Core 預設專案範本

## 前置準備

1. 註冊 IMemoryCache 

    - `Startup.cs` 中的 `ConfigureServices` 方法中加入

        ```cs
        services.AddMemoryCache();
        ```
    
    - 準備 `RegisterPostEvictionCallback`

        ```cs
        private void CacheExpireHandler(object key, object value, EvictionReason reason, object state)
        {
            _logger.LogDebug($"Cache Expire ; key :{key};valu:{value};reason:{reason};state:{state}");
            setCache();
        }
        ```

2. 註冊 NLog

    - 加入 `nlog.config`

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            autoReload="true"
            internalLogLevel="Info"
            internalLogFile="internal-nlog.txt">

            <!-- enable asp.net core layout renderers -->
            <extensions>
                <add assembly="NLog.Web.AspNetCore"/>
            </extensions>

            <!-- the targets to write to -->
            <targets>
                <!-- write logs to file  -->
                <target xsi:type="File" name="allfile" fileName="./logs/nlog-all-${shortdate}.log"
                        layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />

                <!-- another file log, only own logs. Uses some ASP.NET core renderers -->
                <target xsi:type="File" name="ownFile-web" fileName="./logs/nlog-own-${shortdate}.log"
                        layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}|url: ${aspnet-request-url}|action: ${aspnet-mvc-action}" />
            </targets>

            <!-- rules to map from logger name to target -->
            <rules>
                <!--All logs, including from Microsoft-->
                <logger name="*" minlevel="Trace" writeTo="allfile" />

                <!--Skip non-critical Microsoft logs and so log only own logs-->
                <logger name="Microsoft.*" maxlevel="Info" final="true" /> <!-- BlackHole without writeTo -->
                <logger name="*" minlevel="Trace" writeTo="ownFile-web" />
            </rules>
        </nlog>
        ```

    - `Program.cs` 的 `CreateWebHostBuilder` 加入 `.UseNLog()`;

        ```cs
        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                    .UseStartup<Startup>()
                    .UseNLog();
        }
        ```

3. 使用 IMemoryCache 與 NLog

    ```cs
    private static IMemoryCache _cache;
    private ILogger<HomeController> _logger;

    public HomeController(IMemoryCache cache,ILogger<HomeController> logger)
    {
        _cache = cache;
        _logger = logger;
    }
    ```

## 觸發時機

理想狀況就是在 cache expire 當下就觸發 RegisterPostEvictionCallback，即可立馬重新 recache，但實際情況不如預期

1. 確認觸發時機

    ```cs
    public class HomeController : Controller
    {
        private static IMemoryCache _cache;
        private ILogger<HomeController> _logger;

        public HomeController(IMemoryCache cache,ILogger<HomeController> logger)
        {
            _cache = cache;
            _logger = logger;
        }


        public void SetCache()
        {
            _logger.LogDebug($"before SetCache:{JsonConvert.SerializeObject(_cache)}");
            _logger.LogDebug("SetCache Action");
            setCache();
            _logger.LogDebug($"after SetCache:{JsonConvert.SerializeObject(_cache)}");

        }

        public IActionResult Index()
        {
            _logger.LogDebug($"GetCache @ {DateTime.Now}");
            _logger.LogDebug($"before getCache:{JsonConvert.SerializeObject(_cache)}");


            var cacheData = _cache.Get<string>("dateTimeNow");
            _logger.LogDebug($"after getCache:{JsonConvert.SerializeObject(_cache)}");

            return View((object) cacheData);
        }

        public IActionResult Privacy()
        {
            return View();
        }
        
        private void setCache()
        {
            MemoryCacheEntryOptions cacheExpirationOptions = new MemoryCacheEntryOptions();
            var datTimeNow = DateTime.Now;
            _logger.LogDebug($"SetCache @ {datTimeNow}");
            cacheExpirationOptions.AbsoluteExpiration = datTimeNow.AddSeconds(10);
            cacheExpirationOptions.Priority = CacheItemPriority.High;
            cacheExpirationOptions.RegisterPostEvictionCallback(CacheExpireHandler, this);
            _cache.Set<string>("dateTimeNow", datTimeNow.ToString(), cacheExpirationOptions);
        }

        private void CacheExpireHandler(object key, object value, EvictionReason reason, object state)
        {
            _logger.LogDebug($"Cache Expire ; key :{key};valu:{value};reason:{reason};state:{state}");
            setCache();
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel {RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier});
        }
    }
    ```

2. 實際結果

    ![1log](https://user-images.githubusercontent.com/3851540/57983525-54980a00-7a85-11e9-9aa8-9d6c29554db1.png)

## 心得

以測試結果來看， cache 的 expire 機制是被動式的：cache 在無人存取時不會主動刪除，在存取時當下檢查 expire 與否，確定 expire 會連帶刪除 cache 內容接著觸發 `RegisterPostEvictionCallback`

而觸發 `RegisterPostEvictionCallback` 的 cache 當次存取除了會造成刪除 cache 內容還會直接回傳 cache miss 無法取得預期中的 cache 內容，因此光憑 `RegisterPostEvictionCallback` 是沒辦法達成絕對的定期 recache


## 參考資訊

1. [MemoryCache Class](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.caching.memorycache?view=netframework-4.8)
2. [MemoryCache Class](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.caching.memory.memorycache?view=aspnetcore-2.2)
3. [快取在記憶體中的 ASP.NET Core](https://docs.microsoft.com/zh-tw/aspnet/core/performance/caching/memory?view=aspnetcore-2.2)
4. [Getting started with ASP.NET Core 2](https://github.com/NLog/NLog.Web/wiki/Getting-started-with-ASP.NET-Core-2)