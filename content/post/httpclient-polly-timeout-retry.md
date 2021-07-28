---
title: "HttpClient 使用 Polly 做 Timeout 重試"
date: 2021-07-28T21:30:00+08:00
lastmod: 2021-07-28T21:30:00+08:00
draft: false
tags: ["httpclient","csharp","polly"]
slug: "httpclient-polly-timeout-retry"
---

## HttpClient 使用 Polly 做 Timeout 重試

之前使用 HttpClient 做了一個內部的網頁回應偵測工具，原本沒有調整預設的 Timeout 時間 (預設為 100,000 毫秒 = 100 秒，相關說明可以參考 [Microsoft docs:HttpClient.Timeout 屬性](https://docs.microsoft.com/zh-tw/dotnet/api/system.net.http.httpclient.timeout?view=net-5.0&WT.mc_id=DOP-MVP-5002594))，但偵測頻率是 一分鐘 = 60 秒，表示有可能 HttpClient 仍在等待網頁回應時又開啟了下一輪的偵測，於是觀察一段時間的網頁回應都約在 100 毫秒以內，只有在 HttpClient 重新取得 DNS 紀錄時會出現 200-300 毫秒的回應時間，所以初步將 HttpClient Timeout 設定為 3 秒，結果一周內出現好幾次異常：工具偵測到網頁沒有回應，但人工開啟又正常，接著下一次偵測也正常，推測可能是因為該網頁經過層層轉導造成的，所以決定加上 retry 機制，避免 false alarm

不加還好，一加才發現跟我預期的用法有不小落差，趕緊筆記一下避免下次又卡住

## 基本環境說明

1. macOS Big Sur 11.5
2. .NET Core SDK 5.0.202
3. NuGet packages

    - Microsoft.Extensions.Http.Polly 5.0.1

4. 測試 server

    >使用 .NET 5 預設 WebApi 專案範本建立，僅修改 `WeatherForecastController.cs` 將 Get method 在 return 前加上 5 秒的 sleep 模擬超時

    ```cs
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    
    namespace DefaultProject.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        public class WeatherForecastController : ControllerBase
        {
            private static readonly string[] Summaries = new[]
            {
                "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot",     "Sweltering", "Scorching"
            };
    
            private readonly ILogger<WeatherForecastController> _logger;
    
            public WeatherForecastController(ILogger<WeatherForecastController> logger)
            {
                _logger = logger;
            }
    
            [HttpGet]
            public IEnumerable<WeatherForecast> Get()
            {
                var rng = new Random();
                //加上 5 秒 sleep
                Thread.Sleep(5000);

                return Enumerable.Range(1, 5).Select(index => new WeatherForecast
                    {
                        Date = DateTime.Now.AddDays(index),
                        TemperatureC = rng.Next(-20, 55),
                        Summary = Summaries[rng.Next(Summaries.Length)]
                    })
                    .ToArray();
            }
        }
    }
    ```

## 設定方式

1. 移除原始 HttpClient 的 Timeout 設定

    > 如果原本就沒有客製調整，可以忽略

    - 原始設定

        ```cs
        services.AddHttpClient("httpclient_name", c =>
            {
                c.BaseAddress = new Uri("https://localhost:5001/weatherforecast") ;
                c.Timeout = TimeSpan.FromSeconds(3);
            });
        ```

    - 移除設定

        ```cs
        services.AddHttpClient("httpclient_name", c =>
            {
                c.BaseAddress = new Uri("https://localhost:5001/weatherforecast") ;
                // 不設定 HttpClient 的 Timeout 屬性
                //c.Timeout = TimeSpan.FromSeconds(3);
            });
        ```

2. 準備 polly retry policy

    > 設定需要重試的錯誤或條件、重試次數、重試間隔(是否加入 jitter 策略)、每次重試時的額外動作

    ```cs
    private static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
    {
        var jitterier = new Random();
         return HttpPolicyExtensions
                .HandleTransientHttpError()
                // 攔劫 Polly 的 TimeoutPolicy 拋出的 TimeoutRejectedException
                .Or<TimeoutRejectedException>()
                //重試三次，間隔為 1.1 秒的重試次數次方加上 0-20 隨機毫秒
                .WaitAndRetryAsync(3,
                    retryAttempt => TimeSpan.FromSeconds(Math.Pow(1.1, retryAttempt)) +
                                    TimeSpan.FromMilliseconds(jitterier.Next(0, 20))
                    , (exception, timeSpan, retryCount) => { 
                        //紀錄重試的資訊
                        Console.WriteLine($"datetime:{DateTime.Now};timeSpan:{timeSpan};retryCount:{retryCount};exception:{exception.Exception.Message};"); });
    }
    ```

3. 設定 HttpClient 的 polly retry policy 與 timeout handler

    > 將前面設定的 polly retry policy 加至 "httpclient_name" 這個 HttpClient 中，並指定 "httpclient_name" 的 timeout 時間為 3 秒

    ```cs
    services.AddHttpClient("httpclient_name", c =>
        {
            c.BaseAddress = new Uri("https://localhost:5001/weatherforecast") ;
        })
        // 將前面設定的 polly retry policy 加至 "httpclient_name" 這個 HttpClient 中
        .AddPolicyHandler(GetRetryPolicy())
        // 設定 "httpclient_name" 的 timeout 時間為 3 秒
        .AddPolicyHandler(Policy.TimeoutAsync<HttpResponseMessage>(3));
    ```

4. 實際效果：重試三次仍失敗後拋出 `Polly.Timeout.TimeoutRejectedException`

    ![1errorretry](https://user-images.githubusercontent.com/3851540/127306911-2b0fd6da-6093-4036-b92c-977c217c8dc8.png)

- 未移除 Httpclient 的 Timeout 屬性無法由 Polly 正確執行重試策略

    > 直接拋出 `System.TimeoutException`、`System.Threading.Tasks.TaskCanceledException`、`System.IO.IOException`、`System.Net.Sockets.SocketException`，無法被 Polly 攔劫

    ![2timeouterror](https://user-images.githubusercontent.com/3851540/127306952-99b3e15f-5757-49a2-b271-3209a319ee3c.png)

## 心得

雖然我對於為什麼設定了 HttpClient.Timeout 就會造成 Polly 無法正確攔劫 exception 而執行 retry 很感興趣，但因為專案重要所以先筆記一下用法，回頭再花時間仔細了解一下相關機制，之後再回來補充我的心得

## 參考資訊

1. [Microsoft docs:HttpClient.Timeout 屬性](https://docs.microsoft.com/zh-tw/dotnet/api/system.net.http.httpclient.timeout?view=net-5.0&WT.mc_id=DOP-MVP-5002594)
2. [Microsoft docs: 在 ASP.NET Core 中使用託管服務的背景工作](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/host/hosted-services?WT.mc_id=DOP-MVP-5002594&view=aspnetcore-5.0&tabs=visual-studio)
3. [App-vNext/Polly.Extensions.Http](https://github.com/App-vNext/Polly.Extensions.Http/blob/master/README.md)
4. [HttpClient retry on HTTP timeout with Polly and IHttpClientBuilder](https://briancaos.wordpress.com/2020/12/16/httpclient-retry-on-http-timeout-with-polly-and-ihttpclientbuilder/)
