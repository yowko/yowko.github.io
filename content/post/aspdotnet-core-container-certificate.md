---
title: "讓 container 中的 ASP.NET Core 也有憑證"
date: 2020-09-20T21:30:00+08:00
lastmod: 2021-03-04T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC","Docker"]
slug: "aspdotnet-core-container-certificate"
---

## 讓 container 中的 ASP.NET Core 也有憑證

### <span style="color:red"> 經 2021/03/04 測試，本文內容失效，請參考 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) </span>

之前筆記 [使用 ASP.NET Core middleware 進行 gRPC healthy check](/aspdotnet-core-middleware-grpc-healthy-check/)、[使用 ASP.NET Core BackgroundService 進行 gRPC healthy check](/aspdotnet-core-backgroundservice-grpc-healthy-check/) 以及 [ASP.NET Core gRPC 無法在 macOS 上啟動？！](/aspdotnet-core-grpc-macos/) 都有提到過 gRPC 的原生限制：採用 HTTP2 協定並預設使用 SSL，雖然可以使用 clear text 只是如此一來不僅失去安全性在設定上還是比較繁瑣些

與其老是想著要怎麼設定跟調整來避開 gRPC 的 HTTP2 SSL 問題，突然想轉個念：乾脆給個憑證 說不定還簡單些，今天就紀錄一下做法供比較參考囉

## 基本環境說明

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.5(48029)
3. .NET Core SDK 3.1.301
4. docker images

    - yowko/healthcheck:nocert

        > 專案程式碼如下

        - Program.cs

            ```cs
            using Microsoft.AspNetCore.Hosting;
            using Microsoft.Extensions.Hosting;

            namespace HealthCheck_POC
            {
                public class Program
                {
                    public static void Main(string[] args)
                    {
                        CreateHostBuilder(args).Build().Run();
                    }

                    public static IHostBuilder CreateHostBuilder(string[] args) =>
                        Host.CreateDefaultBuilder(args)
                            .ConfigureWebHostDefaults(webBuilder =>
                            {
                               webBuilder.UseStartup<Startup>();
                            });
                }
            }
            ```

        - Startup.cs

            ```cs
            using System;
            using System.Net.Http;
            using Microsoft.AspNetCore.Builder;
            using Microsoft.AspNetCore.Hosting;
            using Microsoft.AspNetCore.Http;
            using Microsoft.Extensions.DependencyInjection;
            using Microsoft.Extensions.Hosting;

            namespace HealthCheck_POC
            {
                public class Startup
                {
                    private IHostApplicationLifetime _hostApplicationLifetime;
                    public void ConfigureServices(IServiceCollection services)
                    {
                        services.AddGrpc();
                        services.AddHealthChecks().AddCheck<GreeterHealthChecker>("example_health_check");
                        //指定 endpoint 為 localhost，並忽略憑證有效性
                        services.AddGrpcClient<Greeter.GreeterClient>(o =>
                        {
                            o.Address = new Uri("https://localhost:5001");
                        })
                            .ConfigurePrimaryHttpMessageHandler(() => {
                                return new HttpClientHandler
                            {
                                ServerCertificateCustomValidationCallback = (message, cert, chain, errors) => true
                            };
                        });
                    }

                    public void Configure(IApplicationBuilder app, IWebHostEnvironment env, IHostApplicationLifetime hostApplicationLifetime)
                    {
                        if (env.IsDevelopment())
                        {
                            app.UseDeveloperExceptionPage();
                        }

                        app.UseRouting();

                        app.UseEndpoints(endpoints =>
                        {
                            endpoints.MapHealthChecks("/health");

                            endpoints.MapGrpcService<GreeterService>();

                            endpoints.MapGet("/",
                                async context =>
                                {
                                    await context.Response.WriteAsync(
                                        "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
                                });
                        });
                    }
                }
            }
            ```

        - appsettings.json

            ```json
            {
              "Logging": {
                "LogLevel": {
                  "Default": "Information",
                  "Microsoft": "Warning",
                  "Microsoft.Hosting.Lifetime": "Information"
                }
              },
              "AllowedHosts": "*",
              "Kestrel": {
                "EndpointDefaults": {
                  "Protocols": "Http2"
                }
              }
            }
            ```

        - GreeterHealthChecker.cs

            ```cs
            using System.Threading;
            using System.Threading.Tasks;
            using Microsoft.Extensions.Diagnostics.HealthChecks;

            namespace HealthCheck_POC
            {
                public class GreeterHealthChecker : IHealthCheck
                {
                    private readonly Greeter.GreeterClient _client;

                    public GreeterHealthChecker(Greeter.GreeterClient client)
                    {
                        _client = client;
                    }

                    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default(CancellationToken))
                    {
                        var reply = await _client.SayHelloAsync(
                        new HelloRequest { Name = "GreeterClient" });

                        if (reply.Message == "Hello GreeterClient")
                        {
                            return HealthCheckResult.Healthy("A healthy result.");
                        }

                        return HealthCheckResult.Unhealthy("An unhealthy result.");
                    }
                }
            }
            ```

        - Protos/greet.proto

            ```protobuf
            syntax = "proto3";

            option csharp_namespace = "HealthCheck_POC";

            package greet;

            // The greeting service definition.
            service Greeter {
              // Sends a greeting
              rpc SayHello (HelloRequest) returns (HelloReply);
            }

            // The request message containing the user's name.
            message HelloRequest {
              string name = 1;
            }

            // The response message containing the greetings.
            message HelloReply {
              string message = 1;
            }
            ```

        - Services/GreeterService.cs

            ```cs
            using System.Threading.Tasks;
            using Grpc.Core;
            using Microsoft.Extensions.Logging;

            namespace HealthCheck_POC
            {
                public class GreeterService : Greeter.GreeterBase
                {
                    private readonly ILogger<GreeterService> _logger;

                    public GreeterService(ILogger<GreeterService> logger)
                    {
                        _logger = logger;
                    }

                    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
                    {
                        return Task.FromResult(new HelloReply
                        {
                            Message = "Hello " + request.Name
                        });
                    }
                }
            }
            ```

        - DOCKERFILE

            ```txt
            FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
            WORKDIR /app
            # 複製 sln csproj and restore nuget
            COPY *.sln .
            # 目標路徑資料夾要與來源路徑資料夾名稱相同(因 sln 已儲存專案路徑)
            COPY HealthCheck_POC/*.csproj ./HealthCheck_POC/
            #RUN dotnet dev-certs https --trust
            # nuget restore
            RUN dotnet restore
            # 複製其他檔案
            COPY HealthCheck_POC/. ./HealthCheck_POC/
            WORKDIR /app/HealthCheck_POC
            # 使用 Release 建置專案並輸出至 out
            RUN dotnet publish -c Release -o out
            FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
            WORKDIR /app
            # 將產出物複製至 app 下
            COPY --from=build /app/HealthCheck_POC/out ./
            # 啟動 TestDokcer
            ENTRYPOINT ["dotnet", "HealthCheck_POC.dll"]
            ```

5. 專案調整與設定

    - 執行指令

        ```bash
        docker run --name poc -p 5001:5001 -e ASPNETCORE_URLS=https://+:5001 yowko/healthcheck:nocert
        ```

    - 錯誤訊息

        ```txt
        crit: Microsoft.AspNetCore.Server.Kestrel[0]
        Unable to start Kestrel.
        System.InvalidOperationException: Unable to configure HTTPS endpoint. No server certificate was specified, and the default developer certificate could not be found or is out of date.
        To generate a developer certificate run 'dotnet dev-certs https'. To trust the certificate (Windows and macOS only) run 'dotnet dev-certs https --trust'.
        For more information on configuring HTTPS see https://go.microsoft.com/fwlink/?linkid=848054.
            at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.UseHttps(ListenOptions listenOptions, Action`1 configureOptions)
            at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.UseHttps(ListenOptions listenOptions)
            at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.AddressesStrategy.BindAsync(AddressBindContext context)
            at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindAsync(IServerAddressesFeature addresses, KestrelServerOptions serverOptions, ILogger logger, Func`2 createBinding)
            at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.StartAsync[TContext](IHttpApplication`1 application, CancellationToken cancellationToken)
        Unhandled exception. System.InvalidOperationException: Unable to configure HTTPS endpoint. No server certificate was specified, and the default developer certificate could not be found or is out of date.
        To generate a developer certificate run 'dotnet dev-certs https'. To trust the certificate (Windows and macOS only) run 'dotnet dev-certs https --trust'.
        For more information on configuring HTTPS see https://go.microsoft.com/fwlink/?linkid=848054.
            at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.UseHttps(ListenOptions listenOptions, Action`1 configureOptions)
            at Microsoft.AspNetCore.Hosting.ListenOptionsHttpsExtensions.UseHttps(ListenOptions listenOptions)
            at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.AddressesStrategy.BindAsync(AddressBindContext context)
            at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.AddressBinder.BindAsync(IServerAddressesFeature addresses, KestrelServerOptions serverOptions, ILogger logger, Func`2 createBinding)
            at Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer.StartAsync[TContext](IHttpApplication`1 application, CancellationToken cancellationToken)
            at Microsoft.AspNetCore.Hosting.GenericWebHostService.StartAsync(CancellationToken cancellationToken)
            at Microsoft.Extensions.Hosting.Internal.Host.StartAsync(CancellationToken cancellationToken)
            at Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync(IHost host, CancellationToken token)
            at Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync(IHost host, CancellationToken token)
            at Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run(IHost host)
            at HealthCheck_POC.Program.Main(String[] args) in /app/HealthCheck_POC/Program.cs:line 18
        ```

    - 錯誤截圖

        ![1error](https://user-images.githubusercontent.com/3851540/93851835-7477b780-fce3-11ea-845a-904f03a9ddef.png)

## 設定步驟

1. 建立開發用憑證

    > 官方文件在此：[使用 Docker over HTTPS 裝載 ASP.NET Core 映射](https://docs.microsoft.com/zh-tw/aspnet/core/security/docker-https?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594)

    - 語法

        ```bash
        dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p {password}
        dotnet dev-certs https --trust
        ```

    - 範例

        ```bash
        dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p pass.123
        dotnet dev-certs https --trust
        ```

2. 將憑證 mount 給 container 並設定 Kestrel 監聽 port

    ```bash
    docker run --name poc -p 5001:5001 -e ASPNETCORE_URLS=https://+:5001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="pass.123" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ yowko/healthcheck:nocert
    ```

3. 實際使用

    > http request 成功觸發 grpc call 並正確取得 grpc response

    ![3request](https://user-images.githubusercontent.com/3851540/93851845-780b3e80-fce3-11ea-8bf1-0f875360993e.png)

    ![2success](https://user-images.githubusercontent.com/3851540/93851844-780b3e80-fce3-11ea-80f8-91dcae487e34.png)

## 心得

1. appsettings.json 中的 `Kestrel` ptotocol 不一定需要指定為 `http2` 使用預設的 `http1andhttp2` 亦可，重點是 certificate
2. `AppContext.SetSwitch("System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);` 無效，需使用 `HttpClientHandler` 設定

反覆測試了幾次，大致上歸納出目前的設定方式供參考

> <span style="color:red"> 經 2021/03/04 測試，本文內容失效，請參考 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) </span>

## 參考資訊

1. [.NET Core 上的 gRPC 簡介](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594)
2. [使用 Docker over HTTPS 裝載 ASP.NET Core 映射](https://docs.microsoft.com/zh-tw/aspnet/core/security/docker-https?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594)
3. [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate)
