---
title: "使用 ASP.NET Core middleware 進行 gRPC healthy check"
date: 2020-08-31T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["asp.net core","gRPC"]
slug: "aspdotnet-core-middleware-grpc-healthy-check"
---

## 使用 ASP.NET Core middleware 進行 gRPC healthy check

這個需求來自於某次的 issue：有個 service 的回應時好時壞，沒有規律，這讓我想起當年在壽險公司資訊部門使用 asp 提供服務的故事，當時也是出現相同的徵兆：雖然兩者相差十幾年、技術也天差地北，但同樣的服務都有時正常有時錯誤

後續 debug 過程中歸納出兩者都有提供多個 serrvice instance：目前專案是透過 kubernetes 設定 3 個 pods，而之前壽險公司是有 8 組 iis 提供服務，時好時壞是因為只有部份 service instance 有問題，之前懷疑過程式版本不一致，但目前系統都是以 image 做為 deploy 的基礎單位，暫時排除，加上重啟該 service instance 後就會正常，不過實際真正造成問題的原因還是沒找到：每次遇到這問題時都是異常已造成使用者問題沒有充足時間慢慢 debug

因為每次遇到這狀況都是 restart service 讓服務恢復正常，所以就想讓重啟這件事變成自動化，而在那之前最重要的就是辨別出異常，立馬來看看可以怎麼做吧

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. Windows 10 version 1809 (OS Build 17763.1397)
3. .NET Core SDK 3.1.401
4. 預設 gRPC Server 專案範本

## 設定方式

1. 調整 proto 產出的程式碼方式為 `Both`

    > 預設為 `Server`

    ![1grpcboth](https://user-images.githubusercontent.com/3851540/91734988-40c2d980-ebde-11ea-8964-c35f653075f3.png)

2. 透過 IHealthCheck 實做 grpc check

    > 這邊透過 di 取得預設範本中的 gRPC client

    ```cs
    public class GreeterHealthChecker : IHealthCheck
    {
        private readonly Greeter.GreeterClient _client;

        public GreeterHealthChecker(Greeter.GreeterClient client)
        {
            _client = client;
        }

        public async Task<HealthCheckResult> CheckHealthAsync(
            HealthCheckContext context,
            CancellationToken cancellationToken = default(CancellationToken))
        {
            //以下就是自訂檢查邏輯
            var reply = await _client.SayHelloAsync(
                new HelloRequest {Name = "GreeterClient"});

            if ( reply.Message == "Hello GreeterClient")
            {
                return HealthCheckResult.Healthy("A healthy result.");
            }

            return  HealthCheckResult.Unhealthy("An unhealthy result.");
        }
    }
    ```

3. 註冊 health check

    > `Startup.cs` 的 `ConfigureServices` 加上 `services.AddHealthChecks();`

    ```cs
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            //....其他需要註冊
            //將上方自訂 health check 方法加至 health check 檢查中
            services.AddHealthChecks().AddCheck<GreeterHealthChecker>("GRPC_health_check");
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
            //將上方自訂 health check 方法加至 health check 檢查中
            services.AddHealthChecks().AddCheck<GreeterHealthChecker>("GRPC_health_check");
            //將本身提供 gRPC 服務的 endpoint 加至 gRPC client factory 中
            services.AddGrpcClient<Greeter.GreeterClient>(o =>
            {
                o.Address = new Uri("https://localhost:5001");
            });
        }
    }
    ```

5. 加上提供查詢 health check 結果的 endpoint

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
    ```

6. 實際效果

    ![2unhealthy](https://user-images.githubusercontent.com/3851540/91734999-43253380-ebde-11ea-9362-7f18a6ce659d.png)

    ![3healthy](https://user-images.githubusercontent.com/3851540/91735000-43bdca00-ebde-11ea-96d3-0363fb334ccc.png)

## 心得

現在團隊的主力開發環境是 macOS，但今天紀錄的這個做法在 macOS 上會失效，錯誤的緣頭是 aspsettings.json 中的 `Kestrel.EndpointDefaults.Protocols="Http2"` 至於原因我猜測與之前筆記 [ASP.NET Core gRPC 無法在 macOS 上啟動？！](/aspdotnet-core-grpc-macos/) 提到的 macOS 不支援具有 TLS 的 ASP.NET Core gRPC 服務有關：只要啟用了 `Kestrel.EndpointDefaults.Protocols="Http2"` ASP.NET Core 中的 http endpoint 就無法正確存取，但這問題這麼大不可能沒人反應呀，我覺得可能是我設定的問題

錯誤畫面如下：

![4error](https://user-images.githubusercontent.com/3851540/91735002-43bdca00-ebde-11ea-93d5-3b6dce66ed2e.png)

相同程式碼在 Windows 下的畫面

![5normal](https://user-images.githubusercontent.com/3851540/91735005-44566080-ebde-11ea-8f6b-e6ccddb430d1.png)

有看到其他做法，這幾日有空再嘗試看看，敬請期待

完整程式碼請參考：[yowko/GrpcHealthCheck](https://github.com/yowko/GrpcHealthCheck)

## 參考資訊

1. [ASP.NET Core gRPC 無法在 macOS 上啟動？！](/aspdotnet-core-grpc-macos/)
2. [ASP.NET Core 中的健康狀態檢查](https://docs.microsoft.com/zh-tw/aspnet/core/host-and-deploy/health-checks?WT.mc_id=DOP-MVP-5002594)
3. [yowko/GrpcHealthCheck](https://github.com/yowko/GrpcHealthCheck)
