---
title: "將 .NET Core Console 專案轉換為 ASP.NET Core"
date: 2019-05-21T19:30:00+08:00
lastmod: 2021-11-03T19:30:31+08:00
draft: false
tags: ["ASP.NET Core","dotnet core"]
slug: "dotnet-core-console-to-aspdotnet-core"
---

## 將 .NET Core Console 專案轉換為 ASP.NET Core

console 因為組成簡單、建立快速，常被用來確認語法或是 POC 特定功能需求，不過一旦 POC 沒問題就會遇到程式碼彙總的狀況，如果程式碼數量眾多，要一隻隻程式檔案慢慢搬至目標專案，不僅容易出錯也很沒效率，身為一名 ~~懶惰~~ 追求高效的開發人員絕對是不允許的，剛好這幾天有個專案有類似需求，順手留個紀錄待日後重新檢視做法 (猜測 .NET Core 3 可能有些不同)

console 專案特性需要使用 `Console.ReadLine()` 或是 `Console.ReadKey()` 來讓程式持續運作，在使用上有些限制，所以打算用 ASP.NET Core 來改善

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. 使用之前筆記 [在 .NET Core console 上使用 Dependency Injection - DI](/dotnet-core-console-di/) 作為基礎來修改
4. NuGet package

    - Microsoft.AspNetCore 2.2.0

## 安裝 NuGet 套件： `Microsoft.AspNetCore`

- 使用 Package Manager

    ```bash
    Install-Package Microsoft.AspNetCore
    ```

- 使用 .NET CLI

    ```bash
    dotnet add package Microsoft.AspNetCore
    ```

## 加入 `Startup.cs`

1. `ConfigureServices` 方法用來設定應用程式的服務
2. `Configure` 方法用來建立應用程式的 request 處理 pipeline

    ```cs
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
        }
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            app.Run(async (context) => { await context.Response.WriteAsync("Hello World!"); });
        }
    }
    ```

## 修改 Host 建立方式

使用 builder pattern 建立 Host 並使用 `Startup` 來設定 service 及 request pipeline

```cs
class Program
{
    static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
           .UseStartup<Startup>();
}
```

## 註冊 DI

修改 `Startup.cs` 的 `ConfigureServices`

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IRunner, YowkoRunner>();
}
```

## 使用方式

修改 `Startup.cs` 的 `Configure`

1. 方法參數加上 `IRunner runner`

2. 使用 `runner` 來執行實際動作

    ```cs
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, IRunner runner)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        app.Run((context) => Task.Run(
            () => runner.DoAction(nameof(Startup))));
    }
    ```

## 實際效果

直接 access 預設的 url

1. <http://localhost:5000>

2. <https://localhost:5001>

即可在 console 看到訊息輸出

![1consoleoutput](https://user-images.githubusercontent.com/3851540/58086674-aa89c080-7bf1-11e9-9463-5a6afc81f302.png)

- 改善console 訊息不明顯

    - 修改 interface 及實作

        ```cs
        public interface IRunner
        {
            string DoAction(string name);
        }
        public class Runner : IRunner
        {
            public string DoAction(string name)
            {
                return $"Doing hard work! {name}";
            }
        }
        public class YowkoRunner : IRunner
        {
            public string DoAction(string name)
            {
                return $"YowkoRunner : Doing hard work! {name}";
            }
        }
        ```
  
    - 修改實際使用

        ```cs
        public void Configure(IApplicationBuilder app, IHostingEnvironment env, IRunner runner)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.Run(async (context) => { await context.Response.WriteAsync(runner.DoAction(nameof(Startup))); });
        }
        ```

        ![2pageoutput](https://user-images.githubusercontent.com/3851540/58086675-aa89c080-7bf1-11e9-9d79-1de86afd2221.png)

## 心得

雖然原本就知道可能會沒什麼營養，但紀錄完發現真的沒什麼內容XD

不過這是下一則筆記的前言，最終目的是紀錄 gRPC 移至 ASP.NET Core 的作法，避免讓 console 升級 ASP.NET Core 內容影與 gRPC 相關設定混在一起才拆開

完成程式碼請參考 [yowko/console2asp.netcore](https://github.com/yowko/console2asp.netcore.git)

## 參考資訊

1. [在 .NET Core console 上使用 Dependency Injection - DI](/dotnet-core-console-di/)
2. [ASP.NET Core 基本概念](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/?view=aspnetcore-2.2&tabs=macos&WT.mc_id=DOP-MVP-5002594)
3. [ASP.NET Core 中的應用程式啟動](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/startup?view=aspnetcore-2.2&WT.mc_id=DOP-MVP-5002594)
