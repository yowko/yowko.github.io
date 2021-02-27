---
title: "ASP.NET Core 避免工作執行到一半強制被關閉"
date: 2021-02-27T09:30:00+08:00
lastmod: 2021-02-27T09:30:31+08:00
draft: false
tags: ["ASP.NET Core"]
slug: "aspdotnetcore-graceful-shutdown"
---

## ASP.NET Core 避免工作執行到一半強制被關閉

最近同事反應部份 application 在不同環境間，有 stop log 沒有正常紀錄下來的情況，雖然只是 log，但很有可能就連原本預期執行完畢的程式碼都沒有順利完成，這樣一來影響範圍可能很大，為了避免情況持續惡化，模擬目前團隊的程式碼結構，找出可能的解決方案，順手紀錄一下

## 基本環境說明

1. macOS Big Sur 11.2.1
2. .NET Core SDK 3.1.406
3. ASP.NET Core WebAPI 預設專案範本

    > 微調 program.cs 中，generic host 的用法：在執行主要程式的前後加上 log

    ```cs
    public static void Main(string[] args)
    {
        var host=CreateHostBuilder(args).Build();
        Console.WriteLine("application start");
        host.Run();
        Console.WriteLine("application stop");
    }
    ```

## 模擬較長的 shutdown 處理

1. 使用 HostedService

    - 建立 HostedService

        > `SlowHostedService.cs`

        ```cs
        public class SlowHostedService: IHostedService
        {
            private readonly ILogger<SlowHostedService> _logger;
    
            public SlowHostedService(ILogger<SlowHostedService> logger)
            {
                _logger = logger;
            }
    
            public Task StartAsync(CancellationToken cancellationToken)
            {
                _logger.LogInformation("SlowHostedService started");
                return Task.CompletedTask;
            }
    
            public async Task StopAsync(CancellationToken cancellationToken)
            {
                _logger.LogInformation($"SlowHostedService stopping..."); 
                await Task.Delay(10_000, cancellationToken);
                _logger.LogInformation("SlowHostedService stopped");
            }
        }
        ```

    - 註冊 HostedService

        > Startup.cs

        ```cs
        public void ConfigureServices(IServiceCollection services)
        {
            //原生註冊，如： services.AddControllers();
            //將執行關閉較耗時的 service 最後註冊，中止時會優先執行
            services.AddHostedService<SlowHostedService>();
        }
        ```

2. 使用 一般 class

    > 這邊偷懶：延用上面的 HostedService，把其註冊成 singleton service ，用來模擬需要較長時間處理關閉作業的動作

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.AddSingleton<SlowHostedService>();
    }
    ```

## 設定方式

1. 使用 HostedService

    > 修改 HostService `ShutdownTimeout` 時間 (預設為 `5秒` [HostOptions.cs](https://github.com/aspnet/Hosting/blob/1da228d28fdfcfb2d77241c4e0413efec1beeafd/src/Microsoft.Extensions.Hosting/HostOptions.cs#L16))

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        //原生註冊，如： services.AddControllers();
        //將執行關閉較耗時的 service 最後註冊，中止時會優先執行
        services.AddHostedService<SlowHostedService>();

        services.Configure<HostOptions>(opts => opts.ShutdownTimeout = TimeSpan.FromSeconds(15));
    }
    ```

    - 修改前

        > HostedService 未執行到 stopped 就拋出錯誤了

        ![2hostedbefore](https://user-images.githubusercontent.com/3851540/109385024-c0322b00-792b-11eb-9ef4-3feec030e7ed.png)

    - 修改後

        > 可以正常執行至 Program.cs 的 "application stop"

        ![3hostedafter](https://user-images.githubusercontent.com/3851540/109385025-c0cac180-792b-11eb-9e3f-365e66cebabe.png)

2. 使用 一般 class

    > 透過 `IHostApplicationLifetime` 註冊 `ApplicationStopped` 事件來進行關閉服務前的處理；這邊也可以直接指定等待時間；簡言之，就是會等待 `life.ApplicationStopped.Register` 區塊都完成後才會往下執行 `host.Run()` 的中止動作

    ```cs
    public static void Main(string[] args)
    {
        var host=CreateHostBuilder(args).Build();
        Console.WriteLine("application start");
        var life = host.Services.GetRequiredService<IHostApplicationLifetime>();
        life.ApplicationStopped.Register(() => {
            Console.WriteLine("Application is shut down");
            // long time works
            var _service = host.Services.GetRequiredService<SlowHostedService>();
            _service.StopAsync(new CancellationToken()).GetAwaiter().GetResult();
            //也可以直接指定等待時間
            //Thread.Sleep(10_1000);
        });
        host.Run();
        Console.WriteLine("application stop");
    }
    ```

    - 修改前

        > 有執行到正常執行至 Program.cs 的 "application stop"，但沒有 service stopped 的訊息，說明並未等待至 service 執行完畢

        ![4applifetimebefore](https://user-images.githubusercontent.com/3851540/109385027-c1635800-792b-11eb-9f58-ca1713bb9848.png)

    - 修改後

        > 有執行到正常執行至 Program.cs 的 "application stop"，也有 service stopped 的訊息

        ![5applifetimeafter](https://user-images.githubusercontent.com/3851540/109385028-c1fbee80-792b-11eb-8b5c-e95efb0e6ab6.png)

## 心得

1. 該用哪種方式？

    > 如果是透過 HostedService 來處理相關作業，選 `放寬 HostService ShutdownTimeout 時間`；如果是其他方式才用 `IHostApplicationLifetime ApplicationStopped 事件`；程式碼會比較簡潔清爽

2. 為什麼是 IHostApplicationLifetime `ApplicationStopped 事件` 而非 `ApplicationStopping`

    >我一開始是用 `ApplicationStopping` 後來發現沒有效果，錯誤如下

    ![1stoppingerror](https://user-images.githubusercontent.com/3851540/109385022-bd373a80-792b-11eb-9fce-be0fe7775a16.png)

3. `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS` 環境變數用法可能已失效

    > 這是在 [Lybecker/k8s-friendly-aspnetcore](https://github.com/Lybecker/k8s-friendly-aspnetcore#graceful-shutdown) 看到的，我實測下並無法成功避免錯誤，據官方文件 [ASP.NET Core Web Host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594#shutdown-timeout) 與 [HostingAbstractionsWebHostBuilderExtensions.UseShutdownTimeout(IWebHostBuilder, TimeSpan) Method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout?WT.mc_id=DOP-MVP-5002594&view=aspnetcore-3.1) 所示，很可能只適用於 ASP.NET Core Web Host

4. 完整程式碼請參考 [yowko/graceful-shutdown-aspdotnetcore](https://github.com/yowko/graceful-shutdown-aspdotnetcore)

## 參考資訊

1. [Extending the shutdown timeout setting to ensure graceful IHostedService shutdown](https://andrewlock.net/extending-the-shutdown-timeout-setting-to-ensure-graceful-ihostedservice-shutdown/)
2. [graceful shutdown asp.net core](https://stackoverflow.com/a/58424476)
3. [HostOptions.cs](https://github.com/aspnet/Hosting/blob/1da228d28fdfcfb2d77241c4e0413efec1beeafd/src/Microsoft.Extensions.Hosting/HostOptions.cs#L16)
4. [yowko/graceful-shutdown-aspdotnetcore](https://github.com/yowko/graceful-shutdown-aspdotnetcore)
5. [ASP.NET Core Web Host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594#shutdown-timeout)
6. [HostingAbstractionsWebHostBuilderExtensions.UseShutdownTimeout(IWebHostBuilder, TimeSpan) Method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout?WT.mc_id=DOP-MVP-5002594&view=aspnetcore-3.1)
