---
title: "個別 HttpClient request 使用不同 Timeout 時間"
date: 2021-08-25T12:30:00+08:00
lastmod: 2021-08-25T12:30:00+08:00
draft: false
tags: ["HttpClient","ASP.NET Core","csharp"]
slug: "httpclient-different-timeout"
---

## 個別 HttpClient request 使用不同 Timeout 時間

在透過 HttpClient 與 partner 介接時，常會使用同一個 named-client 來建立 instance，這也是可以共用 pool 與 存留期設定的推薦方式，雖然可以在 `AddHttpClient` 時加上指定 `Timeout` 的方式來針對該 named-client 設定 Timeout 但如此一來同樣的 named-client instance 都會套用同一個 `Timeout` 設定，如果遇到同個 named-client 但對於不同 request 需要有不同 Timeout 時間就沒辦法滿足，今天就來紀錄一下在同個 named-client instance 設定不同 Timeout 的做法

## 基本環境說明

1. macOS Big Sur 11.5.1
2. .NET Core SDK 5.0.202
3. ASP.NET Core Web Api 預設專案範本

    - server (修改 `WeatherForecastController.cs` 模擬不同 response time)

        ```cs
        [HttpGet("test1")]
        public IEnumerable<WeatherForecast> Get1()
        {
            var rng = new Random();
            Thread.Sleep(500);

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
                {
                    Date = DateTime.Now.AddDays(index),
                    TemperatureC = rng.Next(-20, 55),
                    Summary = Summaries[rng.Next(Summaries.Length)]
                })
                .ToArray();
        }
        
        [HttpGet("test2")]
        public IEnumerable<WeatherForecast> Get2()
        {
            var rng = new Random();
            Thread.Sleep(1500);

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
                {
                    Date = DateTime.Now.AddDays(index),
                    TemperatureC = rng.Next(-20, 55),
                    Summary = Summaries[rng.Next(Summaries.Length)]
                })
                .ToArray();
        }
        
        [HttpGet("test3")]
        public IEnumerable<WeatherForecast> Get3()
        {
            var rng = new Random();
            Thread.Sleep(2500);

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
                {
                    Date = DateTime.Now.AddDays(index),
                    TemperatureC = rng.Next(-20, 55),
                    Summary = Summaries[rng.Next(Summaries.Length)]
                })
                .ToArray();
        }
        ```

    - client (修改 `Startup.cs` 的 `ConfigureServices`)

        ```cs
        services.AddHttpClient( "cts", c =>
                {
                    c.BaseAddress = new Uri("http://localhost:5000/");
                    c.Timeout= TimeSpan.FromSeconds(30);
                }
            );
        ```

## 設定方式

1. 建立 httpclient instance (與一般用法無異)

    ```cs
    private readonly HttpClient _httpClient;

    public WeatherForecastControlle(ILogger<WeatherForecastController> logger,IHttpClientFactoryhttpClientFactory)
    {
        _logger = logger;
        _httpClient = httpClientFactory.CreateClient("cts");
    }
    ```

2. 針對不同 request 使用不同 Timeout：使用 `CancellationTokenSource`

    關於 CancellationTokenSource 可以參考 [[Microsoft Docs] CancellationTokenSource 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.cancellationtokensource?view=net-5.0&WT.mc_id=DOP-MVP-5002594)

    > HttpClient 的 `Timeout` 與 `CancellationTokenSource` 的 Timeout 都設定的情況，會採用時間較短的設定值

    - 程式碼

        ```cs
        var _timeout = 2000;//Timeout 時間(毫秒)
        var cts = new CancellationTokenSource(_timeout);
        var result = await _httpClient.GetAsync($"{request_target}", cts.Token);
        ```

    - 實際案例

        ```cs
        [HttpGet]
        [Route("test1")]
        public async Task<string> Get1()
        {
            var cts1S = new CancellationTokenSource(1000);
            var result = await _httpClient.GetAsync("test1" : target, cts1S.Token);

            return await result.Content.ReadAsStringAsync(new CancellationToken());
        }

        [HttpGet]
        [Route("test2")]
        public async Task<string> Get2()
        {
            var cts2S = new CancellationTokenSource(2000);
            var result = await _httpClient.GetAsync("test2", cts2S.Token);

            return await result.Content.ReadAsStringAsync(new CancellationToken());
        }

        [HttpGet]
        [Route("test3")]
        public async Task<string> Get3()
        {
            var cts3S = new CancellationTokenSource(3000);
            var result = await _httpClient.GetAsync("test3", cts3S.Token);

            return await result.Content.ReadAsStringAsync(new CancellationToken());
        }
        ```

    - 對應的 endpoint 都可以正常服務

        ![1cts1](https://user-images.githubusercontent.com/3851540/130919808-dac7eba5-ea1e-448f-9d61-c8dcb297658b.png)

        ![2cts2](https://user-images.githubusercontent.com/3851540/130919821-50d057f3-b0da-4d93-b2f0-60bceba02fd9.png)

        ![3cts3](https://user-images.githubusercontent.com/3851540/130919825-34051351-877f-4110-93c4-2f081aee09a4.png)

3. 使用短 Timeout 連線長時間 response 模擬斷線

    - 程式碼

        >使用 `1000` 毫秒 timeout 去請求 sleep 2500 的 api (test3)

        ```cs
        [HttpGet
        [Route("test1")]
        public async Task<string> Get1(string target = "")
        {
            var cts1S = new CancellationTokenSource(1000);
            var result = await _httpClient.GetAsync(string.IsNullOrWhiteSpace(target) ? "test1" : target, cts1S.Token);

            return await result.Content.ReadAsStringAsync(new CancellationToken());
        }
        ```

    - 斷線錯誤

        ![4cts1sleep2](https://user-images.githubusercontent.com/3851540/130919830-ff9d403a-40c4-457b-a4b8-bcdb79391730.png)

## 心得

原本我也在懷疑是不是有必要這麼細膩地控制不同 request 的 Timeout 時間，覺得統一設定個 20 秒就夠了吧，如果 20 秒沒辦法回應就以最長的 response 時間為主，但後來遇到 DNS 解析異常，從一開始秒回的 api 就開始卡：統一使用較長 response time 的設定該系統除錯加上了一定難度也錯過最快找到問題的機會

不過針對每個 api 設定不同的 Timeout 也是有風險的：必需持續觀察並評估調整，避免因為上游調整或是異常讓 response time 增加而造成本地系統誤判

至於需不需要針對每個 api 設定 Timeout 還是得看每個系統的特性，站在個人立場是支持的，原因是最小知識原則：單一 api 的 request 原本就不用也不該知道其他 api 的狀況

原始程式碼請參考：[[GitHub]yowko/CancellationTokenSourceForHttpclient](https://github.com/yowko/CancellationTokenSourceForHttpclient)

## 參考資訊

1. [[Microsoft Docs] CancellationTokenSource 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.cancellationtokensource?view=net-5.0&WT.mc_id=DOP-MVP-5002594)
2. [[Microsoft Docs] HttpClient.Timeout 屬性](https://docs.microsoft.com/zh-tw/dotnet/api/system.net.http.httpclient.timeout?view=net-5.0&WT.mc_id=DOP-MVP-5002594)
3. [在C#中使用 CancellationToken 處理非同步任務](https://iter01.com/590833.html)
4. [[GitHub]yowko/CancellationTokenSourceForHttpclient](https://github.com/yowko/CancellationTokenSourceForHttpclient)
