---
title: "使用原生 HostedService 來進行 Schedule Job"
date: 2023-08-01T00:30:00+08:00
lastmod: 2023-08-07T00:30:31+08:00
draft: false
tags: ["csharp","ASP.NET Core"]
slug: "hostedservice-schedule-job"
---

## 使用原生 HostedService 來進行 Schedule Job

最近有個舊專案 (.NET Core 2.1) 需要增加功能，升級 framework 是絕對必要的，只是其中一段用來執行每天更新 cache 的背景作業引起了我的興趣，我忘記為什麼當時沒有引用其他套件 (`Hangfire`，`Quartz.NET`...) 來達成定期執行的目的，但持續以來運作地也滿好的，不過因為需求單純，打算趁著這個機會想要來比較幾個套件與原生做法的優劣，先來紀錄一下 原生 HostedService 的做法

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

    - Cronos 0.7.1

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

1. **非必要** 安裝套件：`Cronos`

    > 這是用來解析 Cron expression 的，可視需要加入

     ```bash
    dotnet add package Cronos --version 0.7.1
    ```

2. 新增 HostedService： `TimedHostedService.cs`

    {{< gist yowko 19f15f7b8ce68f2a423c4abbb5860dd1 >}}

3. 註冊 HostedService：`Program.cs`

    ```cs
    builder.Services.AddHostedService<TimedHostedService>();
    ```

## 心得

我記得之前程式碼是從 microsoft 官方文件抄下來的，但我現在回頭找卻沒找到內容，不知道是我記錯還是 microsoft 官方移除了

- 優點：不用再額外學習其他套件的使用方式
- 缺點：程式碼沒那麼直覺 (我每次看到都要再想一下XD)

原始程式碼在此：[yowko/timedhostedservicedemo](https://github.com/yowko/timedhostedservicedemo)

所有工具使用筆記在此：

1. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
2. [使用 Hangfire 來進行 Schedule Job](/hangfire)
3. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
4. [使用 Coravel 來進行 Schedule Job](/coravel)

## 參考資訊

1. [在 ASP.NET Core 中使用託管服務的背景工作](https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/host/hosted-services?WT.mc_id=DOP-MVP-5002594)
2. [changhuixu/CronJobService.cs](https://gist.github.com/changhuixu/47ffb441575564b57e6446bb59466300)
3. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
4. [使用 Hangfire 來進行 Schedule Job](/hangfire)
5. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
6. [使用 Coravel 來進行 Schedule Job](/coravel)
7. [yowko/timedhostedservicedemo](https://github.com/yowko/timedhostedservicedemo)
