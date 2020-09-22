---
title: "使用 Kubernetes 搭配 ASP.NET Core BackgroundService 確保 gRPC 服務回應合乎預期"
date: 2020-09-22T21:30:00+08:00
lastmod: 2020-09-22T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC","Kubernetes"]
slug: "kubernetes-aspdotnet-core-backgroundservice-grpc"
---

## 使用 Kubernetes 搭配 ASP.NET Core BackgroundService 確保 gRPC 服務回應合乎預期

之前在筆記 [使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期](https://blog.yowko.com/kubernetes-liveness-aspdotnet-core-grpc) 紀錄到使用 Kubernetes Liveness 在 pod 中的 service 回應如果不如預期時就直接重啟 pod 來嘗試恢復 service

而之前另外的筆記 [使用 ASP.NET Core BackgroundService 進行 gRPC healthy check](https://blog.yowko.com/aspdotnet-core-backgroundservice-grpc-healthy-check/) 有不同做法，今天就來紀錄一下 Kubernetes 與 ASP.NET Core BackgroundService 搭配檢查的做法

## 基本環境設定

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.5(48029)
3. Kubernetes v1.16.5
4. .NET Core SDK 3.1.301
5. docker images

    - yowko/healthcheck:bgunhealthy

        > 程式碼與下方 `yowko/healthcheck:bghealthy` 相同，只差在刻意將 gRPC 回應模擬成不同值

    - yowko/healthcheck:bghealthy

        > 專案程式碼如下

        - Program.cs

            ```cs
            using Microsoft.AspNetCore.Hosting;
            using Microsoft.Extensions.Hosting;
            using Microsoft.Extensions.DependencyInjection;

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
                            }).ConfigureServices(services =>
                            {
                                services.AddHostedService<GrpcHealthCheckService>();
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

        - GrpcHealthCheckService.cs

            ```cs
            using System;
            using System.Threading;
            using System.Threading.Tasks;
            using Microsoft.Extensions.Hosting;

            namespace HealthCheck_POC
            {
                public class GrpcHealthCheckService: BackgroundService
                {
                    private readonly Greeter.GreeterClient _client;
                    private IHostApplicationLifetime _hostApplicationLifetime;
                    public GrpcHealthCheckService(Greeter.GreeterClient client,            IHostApplicationLifetime hostApplicationLifetime)
                    {
                        _client = client;
                        _hostApplicationLifetime = hostApplicationLifetime;
                    }
                    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
                    {
                        while (!stoppingToken.IsCancellationRequested)
                        {
                            await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);

                            var reply = await _client.SayHelloAsync(new HelloRequest { Name = "GreeterClient" });
                            //自訂檢查邏輯
                            if (reply.Message != "Hello GreeterClient")
                                _hostApplicationLifetime.StopApplication();
                        }
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

## 設定方式

1. 設定憑證

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

2. 將憑證加至 Kubernetes secret 中

    ```bash
    kubectl create secret generic mysecret --from-file=aspnetapp.pfx=${HOME}/.aspnet/https/aspnetapp.pfx
    ```

3. 將 Kubernetes secret mount 給 pod 做憑證並設定 Liveness

    - bghealthy.yml

        > health check 正常

        ```yml
          apiVersion: v1
          kind: Pod
          metadata:
            name: bgcheckhealthy
          spec:
            volumes:
            - name: secret-volume
              secret:
                secretName: mysecret
            containers:
              - image: yowko/healthcheck:bghealthy
                name: bgcheckhealthy
                ports:
                  - containerPort: 5001
                    protocol: TCP
                volumeMounts:
                - name: secret-volume
                  mountPath: "/https"
                env:
                - name: ASPNETCORE_URLS
                  value: "https://+:5001"
                - name: ASPNETCORE_Kestrel__Certificates__Default__Password
                  value: "pass.123"
                - name: ASPNETCORE_Kestrel__Certificates__Default__Path
                  value: "/https/aspnetapp.pfx"
        ```

    - bgunhealthy.yml

        > 模擬 health check 出現異常

        ```yml
          apiVersion: v1
          kind: Pod
          metadata:
            name: bgcheckunhealthy
          spec:
            volumes:
            - name: secret-volume
              secret:
                secretName: mysecret
            containers:
              - image: yowko/healthcheck:bgunhealthy
                name: bgcheckunhealthy
                ports:
                  - containerPort: 5001
                    protocol: TCP
                volumeMounts:
                - name: secret-volume
                  mountPath: "/https"
                env:
                - name: ASPNETCORE_URLS
                  value: "https://+:5001"
                - name: ASPNETCORE_Kestrel__Certificates__Default__Password
                  value: "pass.123"
                - name: ASPNETCORE_Kestrel__Certificates__Default__Path
                  value: "/https/aspnetapp.pfx"
        ```

4. 使用 `bghealthy.yml` 與 `bgunhealthy.yml` 建立 pod

    ```bash
    kubectl apply -f bghealthy.yml
    ```

    ```bash
    kubectl apply -f bgunhealthy.yml
    ```

5. 實際效果

    > pod `bgcheckunhealthy` 偵測到異常，有重啟 pod 的紀錄，pod `bgcheckhealthy` 則沒有

    ![1podstatus](https://user-images.githubusercontent.com/3851540/93851976-ba348000-fce3-11ea-8284-332bcc8075cc.png)

## 心得

看完內文是不是覺得怪怪的：明明說的是 Kubernetes 搭配 ASP.NET Core BackgroundService 做 gRPC 服務檢查，但實際上檢查是 ASP.NET Core BackgroundService 做的，檢查到異常的終止服務也是 ASP.NET Core 做的，那 Kubernetes 到底做了什麼？！

Kubernetes 在這邊的角色就是確保 ASP.NET Core application pod 正常運行，一旦出現 application 被停止，就會引起 pod status 改變，進而觸發 Kubernetes 重新啟動該 pod

因此 Kubernetes 在與 ASP.NET Core BackgroundService 搭配時，是不用額外做設定的，將檢查的行為與是否重啟的控制權都交給 application，Kubernetes 就負責維持 pod 的 status 即可

## 參考資訊

1. [使用 Docker over HTTPS 裝載 ASP.NET Core 映射](https://docs.microsoft.com/zh-tw/aspnet/core/security/docker-https?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594)
2. [使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期](https://blog.yowko.com/kubernetes-liveness-aspdotnet-core-grpc)
3. [使用 ASP.NET Core BackgroundService 進行 gRPC healthy check](https://blog.yowko.com/aspdotnet-core-backgroundservice-grpc-healthy-check/)
