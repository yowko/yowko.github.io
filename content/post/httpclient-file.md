---
title: "使用 HttpClient 傳送檔案"
date: 2021-07-27T21:30:00+08:00
lastmod: 2021-07-27T21:30:00+08:00
draft: false
tags: ["httpclient","csharp"]
slug: "httpclient-file"
---

## 使用 HttpClient 傳送檔案

最近專案有個需求要將系統畫面 透過 RESTFul API 傳給其他平台做紀錄，這才發現這功能雖然過去待在專案公司時常做，但時間一久覺得好陌生，猛然發現使用 HttpClient 竟然無法直接開工，所以趕緊筆記一下，加深印象，不然總覺得離開發愈來愈遠、快要不會寫程式了 ~~~ 唉

## 基本環境說明

1. macOS Big Sur 11.5
2. .NET Core SDK 5.0.202
3. 測試 server

    >使用 .NET 5 預設 WebApi 專案範本建立，僅修改 `WeatherForecastController.cs` 加入接收 file 參數的 POST method

    ```cs
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
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
                "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm",     "Balmy", "Hot", "Sweltering", "Scorching"
            };
    
            private readonly ILogger<WeatherForecastController> _logger;
    
            public WeatherForecastController(ILogger<WeatherForecastController>     logger)
            {
                _logger = logger;
            }
    
            [HttpGet]
            public IEnumerable<WeatherForecast> Get()
            {
                var rng = new Random();
    
                return Enumerable.Range(1, 5).Select(index => new     WeatherForecast
                    {
                        Date = DateTime.Now.AddDays(index),
                        TemperatureC = rng.Next(-20, 55),
                        Summary = Summaries[rng.Next(Summaries.Length)]
                    })
                    .ToArray();
            }
            
            [HttpPost]
            public async Task Post(IFormFile file)
            {
                if (file.Length > 0)
                {
                    var path =  Path.Combine(Directory.GetCurrentDirectory(), file.FileName);
                    await using var stream = new FileStream(path, FileMode.Create);
                    await file.CopyToAsync(stream);
                }
               
            }
        }
    }
    ```

## 實際程式碼

```cs
var filename = "sample.png";
var url = "https://localhost:5001/weatherforecast";

var formData = new MultipartFormDataContent
{
    {
        // 依檔名將內容讀入 StreamContent
        new StreamContent(new FileStream(filename, FileMode.Open)), "file", filename
    }
};
//指定 post 方法與 post 的 url，並將 StreamContent 指定為 Content
var request = new HttpRequestMessage(HttpMethod.Post, url)
{
    Content = formData
};

using var client = new HttpClient();
var httpResponseMessage = await client.SendAsync(request);
Console.WriteLine(JsonSerializer.Serialize(httpResponseMessage));
```

## 心得

筆記完仔細看了看，一樣覺得陌生XD 再重新回憶一遍才想起過去專案都是後台上傳圖片，好像真的很少透過 RESTFul Api 來傳圖片，搞半天是我記錯了XD

不過查資料的過程中，似乎沒有一致的做法，我猜可能是現在有點規模的系統都是改用微服務，多數情況下都只傳 uri 而已吧，既然這次有這樣的需求加上過去也沒什麼經驗，趁著這個機會筆記一下備忘也是滿好的

## 參考資訊

1. [[鐵人賽 Day23] ASP.NET Core 2 系列 - 上傳/下載檔案](https://blog.johnwu.cc/article/ironman-day23-asp-net-core-upload-download-files.html)
2. [How to upload a file with .NET CORE Web API 3.1 using IFormFile](https://karthiktechblog.com/aspnetcore/how-to-upload-a-file-with-net-core-web-api-3-1-using-iformfile)
3. [(Microsoft Docs)在 ASP.NET Core 上傳檔案](https://docs.microsoft.com/zh-tw/aspnet/core/mvc/models/file-uploads?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
4. [HTTP post file from .NET Core new HTTP client](https://anduin.aiursoft.com/post/2020/4/23/http-post-file-from-net-core-new-http-client)s
