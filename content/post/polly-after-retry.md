---
title: "Polly retry 之後的行為是？"
date: 2019-12-01T21:30:00+08:00
lastmod: 2021-10-29T21:30:31+08:00
draft: false
tags: ["csharp","Library","Polly"]
slug: "polly-after-retry"
---

## Polly retry 之後的行為是？

同事在追查 bug 時，問到在執行某個動作時使用的 Polly policy 來進行失敗重試，如果 policy 中設定的 retry 次數結束仍然失敗會發生什麼事？  印象中之前測試的結果會是直接拋出結果，但時間一久我也忘了XD  於是我就來做個實驗 加深印象吧 ~~

## 基本環境說明

1. macOS Majave 10.14.16
2. .NET Core SDK 3.0.100
3. JetBrains Rider 2019.2.2
4. ASP.NET Core MVC 3.0 預設專案範本

    - Startup.cs

        ```cs
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
            services.AddHttpClient("test",client =>
                {
                    client.BaseAddress = new Uri("http://test.yowko.com");
                    client.Timeout=TimeSpan.FromSeconds(5);
                });
        }
        ```

    - Controller

        ```cs
        [ApiController]
        [Route("[controller]/[action]")]
        public class TestHttpClientController : ControllerBase
        {
            private readonly HttpClient _httpClient;
            public TestHttpClientController(IHttpClientFactory httpClientFactory)
            {
                _httpClient = _httpClientFactory.CreateClient("test");
            }
        }
        ```

5. NuGet packages

    - Polly 7.1.1
    - Microsoft.Extensions.Http.Polly 3.0.0

## 測試內容

測試用 url [http://test.yowko.com](http://test.yowko.com) 沒有實際提供服務

1. 訂定 Polly policy

    > 如果出現 Exception 或是 StatusCode 不是 Success 就進行 retry 3次，每次 retry 會紀錄 retry 時間，並且間隔 1.1 的 retry 次數次方秒

    ```cs
     _pollyPolicy = Policy
        .Handle<Exception>()
        .OrResult<HttpResponseMessage>(a => !a.IsSuccessStatusCode)
        .WaitAndRetryAsync(3, retryAttempt =>
        {
            Console.WriteLine($"polly retry@{DateTime.Now}");
            return TimeSpan.FromSeconds(Math.Pow(1.1, retryAttempt));
        });
    ```

2. 實際使用

    ```cs
    var result= await _pollyPolicy.ExecuteAsync(async () => await _httpClient.GetAsync(""));

    if (result.IsSuccessStatusCode)
        return await result.Content.ReadAsStringAsync();
    return string.Empty;
    ```

3. 實際結果

    > 結果跟印象中一樣：最後一次執行會直接將結果拋出來

    ![1exception](https://user-images.githubusercontent.com/3851540/69914570-cca5aa80-1480-11ea-911d-9c8e0c541aa2.png)

## 攔截可能的 Exception

1. 在 Execute policy 中使用 Try Catch

    ```cs
    var result = await _pollyPolicy.ExecuteAsync(async () =>
        {
            try
            {
                return await _httpClient.GetAsync("");
            }
            catch (Exception e)
            {
                Console.WriteLine($"Exception @ {DateTime.Now}:{e.Message}");
                return new HttpResponseMessage(HttpStatusCode.InternalServerError);
            }
        });

    if (result.IsSuccessStatusCode)
        return await result.Content.ReadAsStringAsync();
    return string.Empty;
    ```

    ![2tryindelegate](https://user-images.githubusercontent.com/3851540/69914571-cca5aa80-1480-11ea-9aa5-f075f3f1419b.png)

2. 將整個 Execute policy 使用 Try Catch 包

    ```cs
    try
    {
        var result = await _pollyPolicy.ExecuteAsync(async () => await _httpClient.GetAsync(""));

        if (result.IsSuccessStatusCode)
            return await result.Content.ReadAsStringAsync();

        return string.Empty;
    }
    catch (Exception e)
    {
        Console.WriteLine($"Exception @ {DateTime.Now}:{e.Message}");

        return string.Empty;
    }
    ```

    ![3tryexecute](https://user-images.githubusercontent.com/3851540/69914572-cd3e4100-1480-11ea-820c-cea798bd0de3.png)

3. 使用 `ExecuteAndCaptureAsync` 方法

    ```cs
     var result= await _pollyPolicy.ExecuteAndCaptureAsync(async () => await _httpClient.GetAsync(""));

    if (result.FinalException==null)
        return await result.Result.Content.ReadAsStringAsync();
    else
        Console.WriteLine($"Exception : {result.ExceptionType} @ {DateTime.Now}:{result.FinalException.Message}");

    return string.Empty;
    ```

    ![4executeandcapture](https://user-images.githubusercontent.com/3851540/69914573-cd3e4100-1480-11ea-88f0-0d6e320a2b0a.png)

    ExecuteAndCaptureAsync 的 result 有下列幾個屬性

    - Outcome - 顯示執行結果
      - 成功 : `Successful`
      - 失敗: `Failure`
    - FinalException - 攔截最後發生的 Exception，如果最終結果是成功這個值會為 null
    - ExceptionType - 最後發生 Exception 的類型，如果最終結果是成功這個值會為 null
      - `Unhandled`:未被 policy 處理
      - `HandledByThisPolicy`: 被 policy 處理
    - Result - 在執行 func 時,如果成功就回傳成功結果，否則就回傳型別的預值

## 心得

三種方式可以依實際需要使用，我個人則是較常使用 `ExecuteAndCaptureAsync`，不過來實際結果來看第二種方式 (將整個 Execute policy 使用 Try Catch 包) 與第三種方式(`ExecuteAndCaptureAsync`) 感覺上差不多，而第一種方式(在 Execute policy 中使用 Try Catch)則比較適用想確實收到每次 exception 的細節，不過需要留意因為多了一層 try catch，在大量使用下的效能損耗

## 參考資訊

1. [App-vNext/Polly](https://github.com/App-vNext/Polly)
