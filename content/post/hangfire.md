---
title: "使用 Hangfire 來進行 Schedule Job"
date: 2023-08-02T00:30:00+08:00
lastmod: 2023-08-02T00:30:31+08:00
draft: false
tags: ["csharp","ASP.NET Core","Tools"]
slug: "hangfire"
---

## 使用 Hangfire 來進行 Schedule Job

最近有個舊專案 (.NET Core 2.1) 需要增加功能，升級 framework 是絕對必要的，只是其中一段用來執行每天更新 cache 的背景作引起了我的興趣，我忘記為什麼當時沒有引用其他套件 (`Hangfire`，`Quartz.NET`...) 來達成定期執行的目的，但持續以來運作地也滿好的，不過因為需求單純，打算趁著這個機會想要來比較幾個套件與原生做法的優劣，今天輪到 Hangfire

需求：

1. 程式啟動先執行一次快取更新
2. 每日 GMT +8  00:00 執行快取更新

所有工具使用筆記在此：

1. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
2. [使用 Hangfire 來進行 Schedule Job](/hangfire)
3. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
4. [使用 Coravel 來進行 Schedule Job](/coravel)

## 基本環境說明

1. macOS Ventura 13.4.1
2. .NET SDK 6.0.400
3. JetBrains Rider 2023.1.4
4. NuGet library

    - Hangfire.Core 1.8.4
    - Hangfire.AspNetCore 1.8.4
    - Hangfire.InMemory 0.5.1

5. 更新 cache 的 code：`UpdateCacheService.cs`

    ```cs
    public class UpdateCacheService
    {
        private readonly ILogger<UpdateCacheService> _logger;
    
        public UpdateCacheService(ILogger<UpdateCacheService> logger)
        {
            _logger = logger;
        }
    
        public Task GetAsync(CancellationToken cancellationToken)
        {
            _logger.LogInformation("Get Data for cache");
            //update cache
            return Task.CompletedTask;
        }
    }
    ```

## 設定步驟

1. 安裝套件

    - Hangfire.Core 1.8.4

        ```bash
        dotnet add package Hangfire.Core --version 1.8.4
        ```

    - Hangfire.AspNetCore 1.8.4

        ```bash
        dotnet add package Hangfire.AspNetCore --version 1.8.4
        ```

    - Hangfire.InMemory 0.5.1

        ```bash
        dotnet add package Hangfire.InMemory --version 0.5.1
        ```

2. 註冊 Hangfire

    - 官方建議

        ```cs
        builder.Services.AddHangfire(configuration => configuration
            .SetDataCompatibilityLevel(CompatibilityLevel.Version_180)
            .UseSimpleAssemblyNameTypeSerializer()
            .UseRecommendedSerializerSettings()
            .UseInMemoryStorage());//使用 memory 來儲存資料
        ```

    - 個人最簡版本

        > 我推測可能是我設定的情境較單純才能用下面這個設定

        ```cs
        builder.Services.AddHangfire(configuration => configuration
            .UseInMemoryStorage());//使用 memory 來儲存資料
        ```

3. 註冊 Hangfire 處理 server 為 IHostedService

    預設 `15秒`會執行一次 job check

    ```cs
    builder.Services.AddHangfireServer();
    ```

    可以透過設定調整 check interval (以下改為 `1秒`)

    ```cs
    builder.Services.AddHangfireServer(options => {
        options.SchedulePollingInterval = TimeSpan.FromSeconds(1);
    });
    ```

4. 設定每日更新快取 job

    ```bash
    const string updateCacheId= "UpdateCache";
    app.Services.GetService<IRecurringJobManager>()
        .AddOrUpdate(updateCacheId, 
            ()=> app.Services.GetService<UpdateCacheService>()!.GetAsync(new CancellationToken()),
            "0 0 * * *",
            new RecurringJobOptions
            {
                TimeZone = TimeZoneInfo.FindSystemTimeZoneById("Asia/Shanghai")
            }
        );
    ```

5. 啟動執行單次更新

    - 方法一：使用 BackgroundJob

        ```cs
        app.Services.GetService<IBackgroundJobClient>().Enqueue<UpdateCacheService>(a => a.GetAsync(new CancellationToken()));
        ```

    - 方法二：手動 trigger 每日更新 job

        ```cs
        app.Services.GetService<IRecurringJobManager>()?.Trigger(updateCacheId);
        ```

## 心得

對 Hangfire 一直有個既定印象：只適合處理分鐘級以上的 job，後來才知道這個因為 job check interval 的關係，仔細想想也合理，這麼簡單的需求想來也不是很難處理，怎麼會做不到XD

- 優點：可以在 Program.cs 中設定完成、設定較多彈性
- 缺點：官網文件沒有更新 miniapi 的版本、文件比較散亂需要東翻西找、nuget library 拆比較多

完整程式碼：[yowko/hangfiredemo](https://github.com/yowko/hangfiredemo)

所有工具使用筆記在此：

1. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
2. [使用 Hangfire 來進行 Schedule Job](/hangfire)
3. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
4. [使用 Coravel 來進行 Schedule Job](/coravel)

## 參考資訊

1. [Hangfire](https://www.hangfire.io/)
2. [Hangfire.InMemory](https://github.com/HangfireIO/Hangfire.InMemory)
3. [ASP.NET Core Library - Hangfire](https://www.cnblogs.com/keatkeat/p/15771877.html)
4. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
5. [使用 Hangfire 來進行 Schedule Job](/hangfire)
6. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
7. [使用 Coravel 來進行 Schedule Job](/coravel)
8. [yowko/hangfiredemo](https://github.com/yowko/hangfiredemo)
