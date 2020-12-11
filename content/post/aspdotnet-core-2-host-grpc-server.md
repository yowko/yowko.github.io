---
title: "使用 ASP.NET Core 2.2 來 Host gRPC Server"
date: 2019-05-21T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC"]
slug: "aspdotnet-core-2-host-grpc-server"
---

# 使用 ASP.NET Core 2.2 來 Host gRPC Server

之前筆記 [Protobuf 該如何處理不定型別](/protobuf-object-any/), [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/), [Protobuf 時間屬性該如何表示？](/protobuf-datetime-timestamp/) 在 host gRPC Server 時都是透過 console project 來進行，但 console 專案需要使用 `Console.ReadLine()` 或是 `Console.ReadKey()` 來讓程式持續運作，實在不太保險，所以打算改用 ASP.NET Core 來 host gRPC Server

ASP.NET Core 將在 .NET Core 3 正式支援 gRPC，其中 gRPC Server host 的部份會透過使用 extension method 的方式在 ConfigureServices 進行註冊，但這個 NuGet package ：[Grpc.AspNetCore.Server](https://www.nuget.org/packages/Grpc.AspNetCore.Server)，目前仍在 preview 階段且會相依於 `.NETCoreApp 3.0`，無法使用在 ASP.NET Core 2.2 上，所幸 gRPC 需要的 HTTP/2 特性在 ASP.NET Core 2.2 中已加入，於是我來筆記一下該怎麼簡易地讓 gRPC Server host 在 ASP.NET Core 2.2 上

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. 使用之前筆記 [在 .NET Core console 上使用 Dependency Injection - DI](/dotnet-core-console-di/) 作為基礎來修改
4. NuGet package

    - Microsoft.AspNetCore 2.2.0

## 升級 NuGet package 並調整建置方式

之前都是透過 `grpc.tools` 的 CLI 工具來將 `.proto` 產生出對應的 C# code，後來經過同事的積極研究後發現從 `grpc.tools 1.17` 開始就支援使用 `dotnet build`、`Visual Studio`、`MSBuild` 來編譯 proto 檔

> 以下操作將以 Rider 示範，Visual Studio 的用法可以直接參考 [Protocol Buffers/gRPC Integration Into .NET Build](https://github.com/grpc/grpc/blob/master/src/csharp/BUILD-INTEGRATION.md)

1. 確認 `Grpc.Tools` 版本比 `1.17` 新

2. 編輯 `gRPC.Message.csproj`

    > include `.proto` ，並加上 `OutputDir` 與 `CompileOutputs`，存檔時就會自動產生 c# 內容
    
    > - `Include` : 指定 `proto` 檔案位置
    > - `OutputDir` : 指定 cs 產出位置，預設為 `/obj/Debug/netstandard2.0` or `/obj/Release/netstandard2.0` 下，視 configuration 而定
    > - `CompileOutputs` : 說明看不懂，但實際上可以避免重複產生 cs 及沒有內容的 cs

    ```xml
    <ItemGroup>
        <Protobuf Include="../../proto/*.proto" OutputDir="%(RelativePath)" CompileOutputs="false"/>
    </ItemGroup>
    ```

## 建立 ASP.NET Core 專案或是將 gRPC.Server 專案改為 ASP.NET Core

以下將用新建 ASP.NET Core 專案 示範，gRPC.Server 專案升級為 ASP.NET Core 詳細做法可以參考之前筆記 [將 .NET Core Console 專案轉換為 ASP.NET Core](/dotnet-core-console-to-aspdotnet-core)

## 使用 ASP.NET Core host gRPC Server

1. 前置準備
    - 將 `gRPC.Message` 專案加入參考
    - 複製 `gRPCServiceImpl.CS`

        ```cs
        public class gRPCServiceImpl : gRPCService.gRPCServiceBase
        {
            public override Task<Response> SayHello(HelloRequest request, ServerCallContext context)
            {
                return Task.FromResult(new Response
                    {
                        IsSuccess = true,
                        ResponseMsg = $"Hi {request.Name} @ {request.SendDate.ToDateTime()} !!!"
                    }
                );
            }

            public override Task<Response> SayGoodbye(GoodByeRequest request, ServerCallContext context)
            {
                return Task.FromResult(new Response
                    {
                        IsSuccess = true,
                        ResponseMsg = $"Bye,{request.Name}"
                    }
                );
            }
        }
        ```

2. 新增 gRPC Server 的啟動 class : `gRPCServer.cs`

    ```cs
    public class gRPCServer
    {
        public string Host { get; private set; }
        public int Port { get; private set; }
        private readonly Grpc.Core.Server serverInstance;

        public gRPCServer(string host, int port, params ServerServiceDefinition[] serverServices)
        {
            Host = host;
            Port = port;
            serverInstance = new Grpc.Core.Server
            //(
            //      // channel 設定請自行依專案狀況調整
            //      new List<ChannelOption>
            //      {
            //          new ChannelOption("grpc.keepalive_permit_without_calls", 1),
            //          new ChannelOption("grpc.http2.max_pings_without_data", 0)
            //      }
            //  )
            {
                Ports =
                {
                    new ServerPort(Host, Port, ServerCredentials.Insecure)
                }
            };
            foreach (var serverService in serverServices)
            {
                serverInstance.Services.Add(serverService);
            }

            serverInstance.Start();
        }
    }
    ```

3. 在 `Startup.cs` 的 `ConfigureServices` 方法中註冊 gRPC

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        var host = "127.0.0.1";
        var port = 50051;

        services.AddSingleton<gRPCService.gRPCServiceBase, gRPCServiceImpl>();

        var Services = services.BuildServiceProvider();
        services.AddSingleton(
            new gRPCServer(host, port,
                gRPCService.BindService(Services.GetRequiredService<gRPCService.gRPCServiceBase>())
            ));
    }
    ```

4. 加入 Jaeger trace

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        ILoggerFactory loggerFactory = new LoggerFactory().AddConsole();
        var serviceName = "gRPC.ServerOnASP.NETCore";
        Tracer tracer = TracingHelper.InitTracer(serviceName, loggerFactory);
        ServerTracingInterceptor tracingInterceptor = new ServerTracingInterceptor(tracer);

        var host = "127.0.0.1";
        var port = 50051;

        services.AddSingleton<gRPCService.gRPCServiceBase, gRPCServiceImpl>();

        var Services = services.BuildServiceProvider();
        services.AddSingleton(
            new gRPCServer(host, port,
                gRPCService.BindService(Services.GetRequiredService<gRPCService.gRPCServiceBase>()).Intercept(tracingInterceptor)
            ));
    }
    ```

## 心得

透過 ASP.NET Core 來 host gRPC Server 除了可以避免在不小心的情況下中止 console 執行，也可以使 configuration、log、cache 這類服務都使用 ASP.NET Core 的 DI,讓開發模式統一，除此之外  等到 .NET Core 3 上市時，轉移至 `Grpc.AspNetCore.Server` 的 effort 也較低

詳細程式碼可以參考 [yowko/asp.netcore2hostgrpc](https://github.com/yowko/asp.netcore2hostgrpc)

## 參考資訊

1. [Protocol Buffers/gRPC Integration Into .NET Build](https://github.com/grpc/grpc/blob/master/src/csharp/BUILD-INTEGRATION.md)