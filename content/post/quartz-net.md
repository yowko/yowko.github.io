---
title: "使用 Quartz.NET 來進行 Schedule Job"
date: 2023-08-03T00:30:00+08:00
lastmod: 2023-08-07T00:30:31+08:00
draft: false
tags: ["csharp","ASP.NET Core","Tools"]
slug: "quartz-net"
---

## 使用 Quartz.NET 來進行 Schedule Job

最近有個舊專案 (.NET Core 2.1) 需要增加功能，升級 framework 是絕對必要的，只是其中一段用來執行每天更新 cache 的背景作業引起了我的興趣，我忘記為什麼當時沒有引用其他套件 (`Hangfire`，`Quartz.NET`...) 來達成定期執行的目的，但持續以來運作地也滿好的，不過因為需求單純，打算趁著這個機會想要來比較幾個套件與原生做法的優劣，今天輪到 Quartz.NET

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

    - Quartz 3.6.3
    - Quartz.AspNetCore 3.6.3

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

1. 安裝 NuGet 套件

    - Quartz

        ```bash
        Install-Package Quartz --version 3.6.3
        ```

    - Quartz.AspNetCore

        ```bash
        Install-Package Quartz.AspNetCore --version 3.6.3
        ```

2. 建立 Job

    {{< gist yowko c95f454b8fe875298742ad4197828614 >}}

3. 註冊 Quartz

    {{< gist yowko 4aca2d8d36a5846fc1f75ee5c7f684f7>}}

4. 將 Quartz 註冊為 HostedService

    ```cs
    builder.Services.AddQuartzHostedService(opt =>
    {
        opt.WaitForJobsToComplete = true;
    });
    ```

## 心得

架構比起其他套件複雜些，原以為有其他套件的經驗，應該是駕輕就熟，想不到使用方式不太一樣，另外 Cron Expression 只能吃六碼覺得不太便民

- 優點：彈性？
- 缺點：架構複雜、文件雜亂

完整程式碼：[yowko/quartz_netdemo](https://github.com/yowko/quartz_netdemo)

所有工具使用筆記在此：

1. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
2. [使用 Hangfire 來進行 Schedule Job](/hangfire)
3. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
4. [使用 Coravel 來進行 Schedule Job](/coravel)

## 參考資訊

1. [Quartz.NET](https://www.quartz-scheduler.net/)
2. [Microsoft DI Integration](https://www.quartz-scheduler.net/documentation/quartz-3.x/packages/microsoft-di-integration.html)
3. [使用原生 HostedService 來進行 Schedule Job](/hostedservice-schedule-job)
4. . [使用 Hangfire 來進行 Schedule Job](/hangfire)
5. [使用 Quartz.NET 來進行 Schedule Job"](/quartz-net)
6. [使用 Coravel 來進行 Schedule Job](/coravel)
7. [yowko/quartz_netdemo](https://github.com/yowko/quartz_netdemo)
