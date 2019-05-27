---
title: "使用 GenericHost 來 Host gRPC Server"
date: 2019-05-26T21:30:00+08:00
lastmod: 2019-05-26T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC","dotnet core"]
slug: "generichost-grpc-server"
---
# 使用 GenericHost 來 Host gRPC Server

之前筆記 [在 .NET Core console 上使用 Dependency Injection - DI](https://blog.yowko.com/dotnet-core-console-di/) 提到 ASP.NET Core 有兩種 host 方式：

1. WebHost
2. Generic Host

而在之前另一則筆記 [使用 ASP.NET Core 2.2 來 Host gRPC Server](https://blog.yowko.com/aspdotnet-core-2-host-grpc-server/) 使用到 WebHost 來 host gRPC Server，經同事提醒其實不需要用到 kestrel 來處理 request，覺得相當有道理於是我就來改版了

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. 使用之前筆記 [使用 ASP.NET Core 2.2 來 Host gRPC Server](https://blog.yowko.com/aspdotnet-core-2-host-grpc-server/) 作為基礎來修改
4. NuGet package

    - Microsoft.AspNetCore 2.2.0
    - Microsoft.Extensions.Hosting 2.2.0

## 修改步驟

1. 建立 .NET Core console 專案
2. 安裝 NuGet package
  
    - Microsoft.AspNetCore 2.2.0
    - Microsoft.Extensions.Hosting 2.2.0

3. 將 `gRPC.Message` 專案加入參考
4. 複製 `gRPCServiceImpl.CS`

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

5. 建立 `gRPCServer.cs` 用來啟動 gRPC Server

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

6. 註冊並啟動 gRPC

    ```cs
    static async Task Main(string[] args)
    {
        var builder = new HostBuilder()
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                //沒有 appsettings.json 就可以不用加
                //config.AddJsonFile("appsettings.json", optional: true);
                config.AddEnvironmentVariables();

                if (args != null)
                {
                    config.AddCommandLine(args);
                }
            })
            .ConfigureServices((hostContext, services) =>
            {
                services.AddOptions();

                var host = "127.0.0.1";
                var port = 50051;

                services.AddSingleton<gRPCService.gRPCServiceBase, gRPCServiceImpl>();
                var Services = services.BuildServiceProvider();
                services.AddSingleton(
                    new gRPCServer(host, port,
                        gRPCService.BindService(Services.GetRequiredService<gRPCService.gRPCServiceBase>())
                    ));
            });

        await builder.RunConsoleAsync();
    }
    ```

7. 加入 Jaeger tracer (optional)

    ```cs
    ILoggerFactory loggerFactory = new LoggerFactory().AddConsole();
    var serviceName = "gRPCServerOnGenericHost";
    Tracer tracer = TracingHelper.InitTracer(serviceName, loggerFactory);
    ServerTracingInterceptor tracingInterceptor = new ServerTracingInterceptor(tracer);

    services.AddSingleton(
    new gRPCServer(host, port,
        gRPCService.BindService(Services.GetRequiredService<gRPCService.gRPCServiceBase>())
            .Intercept(tracingInterceptor)
    ));
    ```

## 心得

Generic Host 適合用來 host 不需處理 http request 的應用程式，以 gRPC Server 而言是相當適合，再次由衷地佩服起同事，深深覺得有強大的同事一起工作真的可以學到很多

回到 ASP.NET Core host gRPC 的主題上，我翻了 `Grpc.AspNetCore.Server` 的原始碼，目前看來主要功能都是使用 WebHost，其中有一部份功能還包含了處理 http request。因此屆時使用 .NET Core 3 ，是不是可以簡單的 migrate 過去可能還需要持續觀察，但 .NET Core 3 還未正式上市，`Grpc.AspNetCore.Server` 也還在 preview，日後會不會調整加入 Generic Host 支援就拭目以待囉

詳細程式碼請參考 [yowko/generichostgrpc](https://github.com/yowko/generichostgrpc)

## 參考資訊

1. [在 .NET Core console 上使用 Dependency Injection - DI](https://blog.yowko.com/dotnet-core-console-di/)
2. [使用 ASP.NET Core 2.2 來 Host gRPC Server](https://blog.yowko.com/aspdotnet-core-2-host-grpc-server/)
3. [ASP.NET Core 基本概念](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/index?view=aspnetcore-2.2&tabs=macos#host)
4. [.NET 泛型主機](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-2.2)