---
title: "ASP.NET Core gRPC 無法在 macOS 上啟動？！"
date: 2019-09-28T20:40:00+08:00
lastmod: 2019-09-28T20:40:30+08:00
draft: false
tags: ["ASP.NET Core","grpc","macOS"]
slug: "aspdotnet-core-grpc-macos"
---

## ASP.NET Core gRPC 無法在 macOS 上啟動？！

隨著 .NET Core 3 正式發表，也代表 gRPC 相關功能開始內建在 ASP.NET Core 3 中並由官方直接支援，前幾個月專案在團隊進行效能評估後已率先於 .NET Core 2.1 起就開始使用 gRPC，當然過程中是踩雷不斷，現在既然有官方正式支援當然就得要來了解相關用法以及找出合適的升級方式，只是沒想到一開始就卡關：使用預設專案範本建立的專案就無法成功執行？！  立馬來看看到底出了什麼事與該如何解決吧

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 3.0.100
3. ASP.NET Core 3 gRPC 預設 server 專案範本

## 錯誤訊息

1. 訊息內容

    ```txt
    warn: Microsoft.AspNetCore.Server.Kestrel[0]
          Unable to bind to https://localhost:5001 on the IPv4 loopback interface: 'HTTP/2 over TLS is not supported on macOS due to missing ALPN support.'.
    warn: Microsoft.AspNetCore.Server.Kestrel[0]
          Unable to bind to https://localhost:5001 on the IPv6 loopback interface: 'HTTP/2 over TLS is not supported on macOS due to missing ALPN support.'.
    crit: Microsoft.AspNetCore.Server.Kestrel[0]
          Unable to start Kestrel.
    System.IO.IOException: Failed to bind to address https://localhost:5001.
     ---> System.AggregateException: One or more errors occurred. (HTTP/2 over TLS is not supported on macOS due to missing ALPN support.) (HTTP/2 over TLS is not  supported on macOS due to missing ALPN support.)
     ---> System.NotSupportedException: HTTP/2 over TLS is not supported on macOS due to missing ALPN support.
       at Microsoft.AspNetCore.Server.Kestrel.Https.Internal.HttpsConnectionMiddleware..ctor(ConnectionDelegate next, HttpsConnectionAdapterOptions options,    ILoggerFactory loggerFactory)
       at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.<>c__DisplayClass12_0.<UseHttps>b__0(ConnectionDelegate next)
       at Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions.Build()
       at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.<>c__DisplayClass21_0`1.<<StartAsync>g__OnBind|0>d.MoveNext()
    --- End of stack trace from previous location where exception was thrown ---
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindEndpointAsync(ListenOptions endpoint, AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.LocalhostListenOptions.BindAsync(AddressBindContext context)
       --- End of inner exception stack trace ---
     ---> (Inner Exception #1) System.NotSupportedException: HTTP/2 over TLS is not supported on macOS due to missing ALPN support.
       at Microsoft.AspNetCore.Server.Kestrel.Https.Internal.HttpsConnectionMiddleware..ctor(ConnectionDelegate next, HttpsConnectionAdapterOptions options,    ILoggerFactory loggerFactory)
       at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.<>c__DisplayClass12_0.<UseHttps>b__0(ConnectionDelegate next)
       at Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions.Build()
       at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.<>c__DisplayClass21_0`1.<<StartAsync>g__OnBind|0>d.MoveNext()
    --- End of stack trace from previous location where exception was thrown ---
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindEndpointAsync(ListenOptions endpoint, AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.LocalhostListenOptions.BindAsync(AddressBindContext context)<---

       --- End of inner exception stack trace ---
       at Microsoft.AspNetCore.Server.Kestrel.Core.LocalhostListenOptions.BindAsync(AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.AddressesStrategy.BindAsync(AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindAsync(IServerAddressesFeature addresses, KestrelServerOptions serverOptions, ILogger  logger, Func`2 createBinding)
       at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.StartAsync[TContext](IHttpApplication`1 application, CancellationToken cancellationToken)
    Unhandled exception. System.IO.IOException: Failed to bind to address https://localhost:5001.
     ---> System.AggregateException: One or more errors occurred. (HTTP/2 over TLS is not supported on macOS due to missing ALPN support.) (HTTP/2 over TLS is not  supported on macOS due to missing ALPN support.)
     ---> System.NotSupportedException: HTTP/2 over TLS is not supported on macOS due to missing ALPN support.
       at Microsoft.AspNetCore.Server.Kestrel.Https.Internal.HttpsConnectionMiddleware..ctor(ConnectionDelegate next, HttpsConnectionAdapterOptions options,    ILoggerFactory loggerFactory)
       at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.<>c__DisplayClass12_0.<UseHttps>b__0(ConnectionDelegate next)
       at Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions.Build()
       at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.<>c__DisplayClass21_0`1.<<StartAsync>g__OnBind|0>d.MoveNext()
    --- End of stack trace from previous location where exception was thrown ---
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindEndpointAsync(ListenOptions endpoint, AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.LocalhostListenOptions.BindAsync(AddressBindContext context)
       --- End of inner exception stack trace ---
     ---> (Inner Exception #1) System.NotSupportedException: HTTP/2 over TLS is not supported on macOS due to missing ALPN support.
       at Microsoft.AspNetCore.Server.Kestrel.Https.Internal.HttpsConnectionMiddleware..ctor(ConnectionDelegate next, HttpsConnectionAdapterOptions options,    ILoggerFactory loggerFactory)
       at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.<>c__DisplayClass12_0.<UseHttps>b__0(ConnectionDelegate next)
       at Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions.Build()
       at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.<>c__DisplayClass21_0`1.<<StartAsync>g__OnBind|0>d.MoveNext()
    --- End of stack trace from previous location where exception was thrown ---
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindEndpointAsync(ListenOptions endpoint, AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.LocalhostListenOptions.BindAsync(AddressBindContext context)<---

       --- End of inner exception stack trace ---
       at Microsoft.AspNetCore.Server.Kestrel.Core.LocalhostListenOptions.BindAsync(AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.AddressesStrategy.BindAsync(AddressBindContext context)
       at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindAsync(IServerAddressesFeature addresses, KestrelServerOptions serverOptions, ILogger  logger, Func`2 createBinding)
       at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.StartAsync[TContext](IHttpApplication`1 application, CancellationToken cancellationToken)
       at Microsoft.AspNetCore.Hosting.GenericWebHostService.StartAsync(CancellationToken cancellationToken)
       at Microsoft.Extensions.Hosting.Internal.Host.StartAsync(CancellationToken cancellationToken)
       at Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync(IHost host, CancellationToken token)
       at Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync(IHost host, CancellationToken token)
       at Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run(IHost host)
       at GrpcService.Program.Main(String[] args) in /Users/yowko.tsai/POCs/DotnetCore3/GrpcService/Program.cs:line 15

    Process finished with exit code 6
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/65829825-fa813e00-e2db-11e9-8cb4-8c917f266678.png)

## 問題原因與解決方式

1. 問題發生原因

    > ASP.NET Core gRPC 範本會使用 TLS 來啟動，而 Kestrel 不支援 macOS 上的 TLS

2. 解決方式

    > 將開發環境的 Kestrel (gRPC Server 端) 與 gRPC Client 端設定使用沒有 TLS 加密的 HTTP/2 協定

    ```cs
    public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            var environmentName = webBuilder.GetSetting("Environment");
            if (string.Compare(environmentName,"Development",StringComparison.InvariantCultureIgnoreCase)==0)
            {
                Console.WriteLine(environmentName);
                webBuilder.ConfigureKestrel(options =>
                {
                    // Setup a HTTP/2 endpoint without TLS.
                    options.ListenLocalhost(5000, o => o.Protocols = HttpProtocols.Http2);
                });
            }
            webBuilder.UseStartup<Startup>();
        });
    ```

## 心得

雖然設定很單純簡單，不過總感覺不夠貼心

另外是關於 `Environment` 的判斷，我沒找到在 `ConfigureWebHostDefaults` 中 `IWebHostEnvironment` 的使用方式，不能只呼叫 `IsDevelopment()` 而免去自行判斷，不知道是我自己學藝不精還是相關處理確實還有改善空間

## 參考資訊

1. [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-3.0#unable-to-start-aspnet-core-grpc-app-on-macos?WT.mc_id=DOP-MVP-5002594)
