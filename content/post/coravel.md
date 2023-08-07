---
title: "使用 Coravel 來進行 Schedule Job"
date: 2023-08-04T00:30:00+08:00
lastmod: 2023-08-07T00:30:31+08:00
draft: false
tags: ["csharp","ASP.NET Core","Tools"]
slug: "coravel"
---

## 使用 Coravel 來進行 Schedule Job

最近有個舊專案 (.NET Core 2.1) 需要增加功能，升級 framework 是絕對必要的，只是其中一段用來執行每天更新 cache 的背景作業引起了我的興趣，我忘記為什麼當時沒有引用其他套件 (`Hangfire`，`Quartz.NET`...) 來達成定期執行的目的，但持續以來運作地也滿好的，不過因為需求單純，打算趁著這個機會想要來比較幾個套件與原生做法的優劣，今天輪到 Coravel

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

    - Coravel 4.2.1

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

1. 安裝 `Coravel`

    ```bash
    dotnet add package coravel --version 4.2.1
    ```

2. 註冊 Coravel： `Program.cs`

    ```cs
    builder.Services.AddScheduler();
    ```

3. 設定排程

    - 方法 1：使用 `Invocables` (官方推薦)

        - a. 建立 `Invocables`：`UpdateCacheJob.cs`

            {{< gist yowko 74a82ed9f06ab52730f176a3029d0b56 >}}

        - b. 註冊 `Invocables`

            > scope 是 `Transient`，但官網上沒有說明原因，我用 Singleton 也正常

            ```cs
            builder.Services.AddTransient<UpdateCacheJob>();
            ```

        - c. 設定 `Invocables` 排程

            {{< gist yowko 964b4bd0b4a1fcfc8da2b80c496452d2 >}}

    - 方法 2：直接呼叫方法

        {{< gist yowko db60edaeb8f13150d7d2a454d7fef829 >}}

## 心得

Coravel 的官網文件還沒更新到 .NET 6 的版本，說明也略少，但大致上簡潔明瞭

- 優點：設定上很直覺、文件說明簡單明瞭
- 缺點：預設會吞掉 exception，也無法 re-throw、文件說明簡單明瞭XD (有的地方真的簡單到不知道為什麼)

完整程式碼：[yowko/coraveldemo](https://github.com/yowko/coraveldemo)

所有工具使用筆記在此：

1. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
2. [使用 Hangfire 來進行 Schedule Job](/hangfire)
3. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
4. [使用 Coravel 來進行 Schedule Job](/coravel)

## 參考資訊

1. [Coravel](https://docs.coravel.net/)
2. [[NETCore] ASP.NET Core 中的排程利器 - Coravel](https://marcus116.blogspot.com/2019/09/task-schedule-library-coravel-in-netcore-aspnetcore.html)
3. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
4. [使用 Hangfire 來進行 Schedule Job](/hangfire)
5. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
6. [使用 Coravel 來進行 Schedule Job](/coravel)
7. [yowko/coraveldemo](https://github.com/yowko/coraveldemo)
