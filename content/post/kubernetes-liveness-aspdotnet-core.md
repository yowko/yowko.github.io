---
title: "使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期"
date: 2020-09-21T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC","Kubernetes"]
slug: "kubernetes-liveness-aspdotnet-core-grpc"
---

## 使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期

今天要紀錄透過 Kubernetes 搭配 [使用 ASP.NET Core middleware 進行 gRPC healthy check](/aspdotnet-core-middleware-grpc-healthy-check/) (當然 [使用 ASP.NET Core BackgroundService 進行 gRPC healthy check](/aspdotnet-core-backgroundservice-grpc-healthy-check/) 也是可行的) 與 [讓 container 中的 ASP.NET Core 也有憑證](/aspdotnet-core-container-certificate) 來確保 service 的回應正確

## 基本環境設定

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.5(48029)
3. Kubernetes v1.16.5
4. .NET Core SDK 3.1.301
5. docker images

    - yowko/healthcheck:unhealthy

        > 程式碼與下方 `yowko/healthcheck:healthy` 相同，只差在刻意將 gRPC 回應模擬成不同值

    - yowko/healthcheck:healthy

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

    - healthy.yml

        > health check 正常

        ```yml
          apiVersion: v1
          kind: Pod
          metadata:
            name: checkhealthy
          spec:
            volumes:
            - name: secret-volume
              secret:
                secretName: mysecret
            containers:
              - image: yowko/healthcheck:healthy
                name: checkhealthy
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
                livenessProbe:
                  httpGet:
                    path: /health
                    port: 5001
                    scheme: HTTPS
                  initialDelaySeconds: 30
        ```

    - unhealthy.yml

        > 模擬 health check 出現異常

        ```yml
          apiVersion: v1
          kind: Pod
          metadata:
            name: checkunhealthy
          spec:
            volumes:
            - name: secret-volume
              secret:
                secretName: mysecret
            containers:
              - image: yowko/healthcheck:unhealthy
                name: checkunhealthy
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
                livenessProbe:
                  httpGet:
                    path: /health
                    port: 5001
                    scheme: HTTPS
                  initialDelaySeconds: 30
        ```

4. 使用 `healthy.yml` 與 `unhealthy.yml` 建立 pod

    ```bash
    kubectl apply -f healthy.yml
    ```

    ```bash
    kubectl apply -f unhealthy.yml
    ```

5. 實際效果

    > pod `checkunhealthy` 偵測到異常，有重啟 pod 的紀錄，pod `checkhealthy` 則沒有

    ![1podstatus](https://user-images.githubusercontent.com/3851540/93851931-a1c46580-fce3-11ea-9850-a2b5894ea747.png)

## 心得

今天是透過 `Kubernetes Liveness Probe` 來確保服務回應如預期，主要是希望如果服務出現回應不如預期時直接進行重啟 (過去經驗服務重啟有機會可以排除狀況)，而不是透過 `Kubernetes Readiness Probe` 僅將該 pod 下線(不提供服務)，但這個做法是我所處團隊的選擇不一定適用於所有情境

除了今天提到的做法之外，也可以將服務停止的控制權交給 application，詳細內容可以參考 [使用 Kubernetes 搭配 ASP.NET Core BackgroundService 確保 gRPC 服務回應合乎預期](/kubernetes-aspdotnet-core-backgroundservice-grpc)

## 參考資訊

1. [使用 ASP.NET Core middleware 進行 gRPC healthy check](/aspdotnet-core-middleware-grpc-healthy-check/)
2. [使用 ASP.NET Core BackgroundService 進行 gRPC healthy check](/aspdotnet-core-backgroundservice-grpc-healthy-check/)
3. [讓 container 中的 ASP.NET Core 也有憑證](/aspdotnet-core-container-certificate)
4. [使用 Docker over HTTPS 裝載 ASP.NET Core 映射](https://docs.microsoft.com/zh-tw/aspnet/core/security/docker-https?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594)
5. [Kubernetes storing persistent files](https://stackoverflow.com/questions/50958650/kubernetes-storing-persistent-files)
6. [Define Environment Variables for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
7. [Kubernetes - Health Check](https://tn710617.github.io/zh-tw/config-liveness-readiness-startup-probes/)
8. [Kubernetes and Containers Best Practices - Health Probes](https://www.magalix.com/blog/kubernetes-and-containers-best-practices-health-probes)
9. [使用 Kubernetes 搭配 ASP.NET Core BackgroundService 確保 gRPC 服務回應合乎預期](/kubernetes-aspdotnet-core-backgroundservice-grpc)
