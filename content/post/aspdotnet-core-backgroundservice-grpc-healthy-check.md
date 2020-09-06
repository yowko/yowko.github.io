---
title: "使用 ASP.NET Core BackgroundService 進行 gRPC healthy check"
date: 2020-09-06T21:30:00+08:00
lastmod: 2020-09-06T21:30:31+08:00
draft: false
tags: ["asp.net core","gRPC"]
slug: "aspdotnet-core-backgroundservice-grpc-healthy-check"
---

## 使用 ASP.NET Core BackgroundService 進行 gRPC healthy check

之前筆記 [使用 ASP.NET Core middleware 進行 gRPC healthy check](https://blog.yowko.com/aspdotnet-core-middleware-grpc-healthy-check) 紀錄到如何使用 ASP.NET Core 內建的 Health Check middleware 來進行 gRPC service 的檢查，當時有看到 Steve Gordon 的 [HEALTH CHECKS WITH GRPC AND ASP.NET CORE 3.0](https://www.stevejgordon.co.uk/health-checks-with-grpc-and-asp-net-core-3) 使用 serive 本身的 BackgroundService 來進行 gRPC Health Check，雖然目的不太一樣，但做法還是可以參考的，簡單紀錄一下

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. Windows 10 version 1809 (OS Build 17763.1397)
3. .NET Core SDK 3.1.401
4. 預設 gRPC Server 專案範本

## 設定方式

1. 調整 proto 產出的程式碼方式為 `Both`

    > 預設為 `Server`

    ![1grpcboth](https://user-images.githubusercontent.com/3851540/91734988-40c2d980-ebde-11ea-8964-c35f653075f3.png)

2. 建立 BackgroundService 實做 grpc check

    > 這邊透過 di 取得預設範本中的 gRPC client

    ```cs
    public class GrpcHealthCheckService : BackgroundService
    {
        private readonly Greeter.GreeterClient _client;
        public GrpcHealthCheckService(Greeter.GreeterClient client)
        {
            _client = client;
        }
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                var reply = await _client.SayHelloAsync(new HelloRequest { Name = "GreeterClient" });
                Console.WriteLine(reply.Message);
                //自訂檢查邏輯
                if (reply.Message == "Hello GreeterClient")
                    HealthModel.HealthStatus = "Healthy";
                else
                    HealthModel.HealthStatus = "Unhealthy";
                await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);
            }
        }
    }
    ```

3. 註冊 BackgroundService

    > `Startup.cs` 的 `ConfigureServices` 加上 `services.AddHealthChecks();`

    ```cs
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            //....其他需要註冊
            //註冊上方自訂 BackgroundService
            services.AddHostedService<GrpcHealthCheckService>();
        }
    }
    ```

4. 註冊 gRPC client

    > `Startup.cs` 的 `ConfigureServices` 加上 `services.AddGrpcClient();`

    ```cs
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            //....其他需要註冊
            //註冊上方自訂 BackgroundService
            services.AddHostedService<GrpcHealthCheckService>();
            //將本身提供 gRPC 服務的 endpoint 加至 gRPC client factory 中
            services.AddGrpcClient<Greeter.GreeterClient>(o =>
            {
                o.Address = new Uri("https://localhost:5001");
            });
        }
    }
    ```

5. 加上提供提供 health check 結果的 endpoint

    > `Startup.cs` 的 `Configure` 加上 `endpoints.MapHealthChecks("/health");`

    ```cs
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            // 提供查詢 health checks 結果的 endpoint
            endpoints.MapGet("/health",
                    async context=> {
                        await context.Response.WriteAsync($"{HealthModel.HealthStatus}");
                    });

            endpoints.MapGrpcService<GreeterService>();

            endpoints.MapGet("/",
                async context =>
                {
                    await context.Response.WriteAsync(
                        "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
                });
        });
    }
    ```

6. 實際效果

    ![2unhealthy](https://user-images.githubusercontent.com/3851540/91734999-43253380-ebde-11ea-9362-7f18a6ce659d.png)

    ![3healthy](https://user-images.githubusercontent.com/3851540/91735000-43bdca00-ebde-11ea-96d3-0363fb334ccc.png)

## 心得

今天這個做法與之前筆記 [使用 ASP.NET Core middleware 進行 gRPC healthy check](https://blog.yowko.com/aspdotnet-core-middleware-grpc-healthy-check) 在 macOS 上有相同問題：現在團隊的主力開發環境是 macOS，但今天紀錄的這個做法在 macOS 上會失效，錯誤的緣頭是 aspsettings.json 中的 `Kestrel.EndpointDefaults.Protocols="Http2"` 至於原因我猜測與之前筆記 [ASP.NET Core gRPC 無法在 macOS 上啟動？！](https://blog.yowko.com/aspdotnet-core-grpc-macos/) 提到的 macOS 不支援具有 TLS 的 ASP.NET Core gRPC 服務有關：只要啟用了 `Kestrel.EndpointDefaults.Protocols="Http2"` ASP.NET Core 中的 http endpoint 就無法正確存取，但這問題這麼大不可能沒人反應呀，我覺得可能是我設定的問題

錯誤畫面如下：

![4error](https://user-images.githubusercontent.com/3851540/91735002-43bdca00-ebde-11ea-93d5-3b6dce66ed2e.png)

相同程式碼在 Windows 下的畫面

![5normal](https://user-images.githubusercontent.com/3851540/91735005-44566080-ebde-11ea-8f6b-e6ccddb430d1.png)

完整程式碼請參考：[yowko/GrpcHealthCheck2](https://github.com/yowko/GrpcHealthCheck2)

## 參考資訊

1. [ASP.NET Core gRPC 無法在 macOS 上啟動？！](https://blog.yowko.com/aspdotnet-core-grpc-macos/)
2. [使用 ASP.NET Core middleware 進行 gRPC healthy check](https://blog.yowko.com/aspdotnet-core-middleware-grpc-healthy-check)
3. [HEALTH CHECKS WITH GRPC AND ASP.NET CORE 3.0](https://www.stevejgordon.co.uk/health-checks-with-grpc-and-asp-net-core-3)
4. [Background tasks with hosted services in ASP.NET Core](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/host/hosted-services?WT.mc_id=DOP-MVP-5002594)
5. [Implement background tasks in microservices with IHostedService and the BackgroundService class](https://docs.microsoft.com/zh-tw/dotnet/architecture/microservices/multi-container-microservice-net-applications/background-tasks-with-ihostedservice?WT.mc_id=DOP-MVP-5002594)
6. [yowko/GrpcHealthCheck2](https://github.com/yowko/GrpcHealthCheck2)
