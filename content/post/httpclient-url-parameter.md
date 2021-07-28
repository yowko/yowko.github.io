---
title: "HttpClient 使用 URL Parameters (Query Strings) 的方式"
date: 2021-07-24T21:30:00+08:00
lastmod: 2021-07-27T21:30:00+08:00
draft: false
tags: ["httpclient","csharp"]
slug: "httpclient-url-parameter"
---

## HttpClient 使用 URL Parameters (Query Strings) 的方式

前陣子有個跟其他系統介接的需求，因為手上工作太滿，先由前端的同事使用 Node.js 開發，過陣子再接手改用 C# 開發；其中有個功能需要 post 幾個參數，一直沒能成功，仔細翻查同事的 source code 才發現這個功能雖然使用 post method 傳遞參數，但傳遞的途徑不是透過 body 而是 url，而這樣的做法在 Node.js 中可以使用 `URLSearchParams` 的 API 來處理，這打醒了過去一直自行組裝 url 的我，於是我便嘗試了一下在 HttpClient 中較結構化處理 URL Parameters 的做法，快速筆記一下備忘

## 基本環境說明

1. macOS Big Sur 11.5
2. .NET Core SDK 5.0.202
3. NuGet packages

    - Microsoft.AspNetCore.WebUtilities 2.2.0

4. 測試 server

    >使用 .NET 5 預設 WebApi 專案範本建立，僅修改 `WeatherForecastController.cs` 加入接收 Query 參數的 POST method

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
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
                "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm",     "Balmy", "Hot", "Sweltering", "Scorching"
            };
    
            private readonly ILogger<WeatherForecastController> _logger;
    
            public WeatherForecastController(ILogger<WeatherForecastController>     logger)
            {
                _logger = logger;
            }
            
            [HttpPost]
            public void Post([FromQuery] string id,[FromQuery] string name,    [FromQuery]int age)
            {
               _logger.LogInformation($"id:{id}");
               _logger.LogInformation($"name:{name}");
               _logger.LogInformation($"age:{age}");
               
            }
        }
    }
    ```

## 使用方式

1. 自行組裝 url

    - 程式碼

        ```cs
        class Program
        {
            private static readonly System.Net.Http.HttpClient client = new System.Net.Http.HttpClient();
            static async Task Main(string[] args)
            {
                var id = 1;
                var name = "yowko";
                var age = 20;

                var result = await client.PostAsync("https://localhost:5001/weatherforecast?id={id}&name={name}&age={age}",null);
                Console.WriteLine(result);
            }
        }
        ```

    - server 結果

        ![1assmebleurl](https://user-images.githubusercontent.com/3851540/126904292-efefbd46-8bc2-49d1-8cd6-b4d653ee47cb.png)

2. 使用 `Microsoft.AspNetCore.WebUtilities` 的 `QueryHelpers`

    - 安裝 NuGet 套件：`Microsoft.AspNetCore.WebUtilities`
    - 程式碼

        ```cs
        class Program
        {
            private static readonly System.Net.Http.HttpClient client = new System.Net.Http.HttpClient();

            static async Task Main(string[] args)
            {
                var id = 1;
                var name = "yowko";
                var age = 20;

                var values = new Dictionary<string, string>
                {
                    {"id", $"{id}_query"},
                    {"name", $"{name}_query"},
                    {"age", $"{age + 1}"}
                };

                var requestUri = QueryHelpers.AddQueryString("https://localhost:5001/weatherforecast", values);
                var request = new HttpRequestMessage(HttpMethod.Post,requestUri);

                var result = await client.SendAsync(request);
                Console.WriteLine(result);
            }
        }
        ```

    - server 結果

        ![2webutilities](https://user-images.githubusercontent.com/3851540/126904298-29ecf29a-c486-4646-8528-233ca7ee7758.png)

## 心得

就範例的程式碼來看，自行組裝 url 程式碼數量還比較少，但如果需要處理的參數較多或是日後維護性來看，我個人會偏好使用 `QueryHelpers`，出發點是一遇到需要調整參數與條件時，都要慢慢比對那個組裝出來 url，一來怕看錯、看漏，二來也覺得雜亂，不過這還是個人選擇只要能達成需求就是好程式，今天就紀錄一下不同做法供參考囉

## 參考資訊

1. [What Are URL Parameters (Query Strings)?](https://www.kameronjenkins.com/seo/url-parameters-query-strings)
2. [在 ASP.NET Core 中使用 IHttpClientFactory 發出 HTTP 要求](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/http-requests?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
3. [QueryHelpers.AddQueryString 方法](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.aspnetcore.webutilities.queryhelpers.addquerystring?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
