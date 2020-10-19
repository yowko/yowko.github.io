---
title: "ASP.NET Core 的 gRPC Polly Retry"
date: 2020-10-19T21:30:00+08:00
lastmod: 2020-10-19T21:30:31+08:00
draft: false
tags: ["asp.net core","gRPC"]
slug: "aspdotnet-core-grpc-polly-retry"
---

## ASP.NET Core 的 gRPC Polly Retry

之前曾經對 gRPC 的 keepalive 調整一輪 (server、kubernetes、container)，原本看似已解決，但最近斷線的錯誤又出現，於是改變想法：乾脆直接做 retry 重連

## 基本環境說明

1. macOS Catalina 10.15.7
2. .NET Core SDK 3.1.301
3. ASP.NET Core gRPC 預設專案範本

## 設定方式

1. 定義連線狀態

    > 大部份情況下，gRPC 都會以 http status code 回應，但在 gRPC 專屬情境下則會回出 grpc-status，所以兩者皆需要處理

    - http

        ```cs
        var httpErrors = new HttpStatusCode[] {
                 HttpStatusCode.BadGateway,
                 HttpStatusCode.GatewayTimeout,
                 HttpStatusCode.ServiceUnavailable,
                 HttpStatusCode.InternalServerError,
                 HttpStatusCode.TooManyRequests,
                 HttpStatusCode.RequestTimeout
        };
        ```

    - gRPC

        ```cs
        var grpcErrors = new StatusCode[] {
                 StatusCode.DeadlineExceeded,
                 StatusCode.Internal,
                 StatusCode.NotFound,
                 StatusCode.ResourceExhausted,
                 StatusCode.Unavailable,
                 StatusCode.Unknown
        };
        ```

2. 取得 grpc-status

    ```cs
    public static class StatusManager
    {
        public static StatusCode? GetStatusCode(HttpResponseMessage response)
        {
            var headers = response.Headers;

            if (!headers.Contains("grpc-status") && response.StatusCode == HttpStatusCode.OK)
                return StatusCode.OK;

            if (headers.Contains("grpc-status"))
                return (StatusCode)int.Parse(headers.GetValues("grpc-status").First());

            return null;
        }
    }
    ```

3. Polly retry policy

    ```cs
    Func<HttpRequestMessage, IAsyncPolicy<HttpResponseMessage>> retryFunc = request =>
    {
        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .OrResult(
                r => {

                var grpcStatus = StatusManager.GetStatusCode(r);
                var httpStatusCode = r.StatusCode;

                // `出現 http 錯誤` 或是 `出現 grpc 錯誤 (grpc 會回應 HttpStatusCode.OK)`
                return (grpcStatus == null && serverErrors.Contains(httpStatusCode)) || (httpStatusCode == HttpStatusCode.OK && gRpcErrors.Contains(grpcStatus.Value));
                }
            )
            //重試 `1` 次，間隔 `1` 秒
            .WaitAndRetryAsync(1,
                (input) => TimeSpan.FromSeconds(1),
                (result, timeSpan, retryCount, context) =>
                {
                    //log 重試次數
                    Log.Warning($"Retry:${retryCount}");
                });
            };
    ```

4. 使用 Polly policy 加入 gRPCClientFactory

    ```cs
    services.AddGrpcClient<Greeter.GreeterClient>(o =>
        {
            o.Address = new Uri("https://localhost:5001");
        })
        .AddPolicyHandler(retryFunc);
    ```

## 心得

如果只是為了解決 gRPC keepalive 的問題， retry 一次用來觸發 gRPC 重新連線我個人覺得最為理想，但若是不僅僅要處理 gRPC keepalive 問題就得要自行調整 retry 次數與間隔時間，甚至加上 jitter 策略了

## 參考資訊

1. [gRPC & ASP.NET Core 3.1: Resiliency with Polly](https://anthonygiretti.com/2020/03/31/grpc-asp-net-core-3-1-resiliency-with-polly/)
2. [使用 HttpClientFactory 實作復原 HTTP 要求](https://docs.microsoft.com/zh-tw/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests?WT.mc_id=DOP-MVP-5002594)
3. [實作斷路器模式](https://docs.microsoft.com/zh-tw/dotnet/architecture/microservices/implement-resilient-applications/implement-circuit-breaker-pattern?WT.mc_id=DOP-MVP-5002594)
