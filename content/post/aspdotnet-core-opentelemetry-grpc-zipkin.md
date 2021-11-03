---
title: "[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫"
date: 2021-04-04T22:30:00+08:00
lastmod: 2021-04-04T22:30:31+08:00
draft: false
tags: ["Zipkin","OpenTelemetry","gRPC","ASP.NET Core"]
slug: "aspdotnet-core-opentelemetry-grpc-zipkin"
---

## [Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫

之前筆記 [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-zipkin) 紀錄到使用 OpenTelemetry 搭配 Zipkin 來追蹤 ASP.NET Core 也曾在 [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/) 紀錄使用 OpenTracing 來追蹤 gRPC 呼叫，之前筆記 [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](aspdotnet-core-opentelemetry-grpc-jaeger) 已經紀錄了 OpenTelemetry 搭配 Jaeger 追蹤 gRPC，今天就快速紀錄一下如何使用 OpenTelemetry 搭配 Zipkin 來 追蹤 ASP.NET Core 上的 gRPC 吧

## 基本環境說明

1. macOS Big Sur 11.2.3
2. docker desktop 3.2.2 (61853)

    - openzipkin/zipkin-slim:2

3. NuGet packages

    - GrpcService
        - Google.Protobuf 3.15.7
        - Grpc.Tools 2.36.4
        - Grpc.Core 2.36.4
    - Zipkin_gRPCClient
        - Grpc.Net.Client 2.36.0
        - Microsoft.Extensions.Configuration.Json 5.0.0
        - OpenTelemetry.Instrumentation.GrpcNetClient 1.0.0-rc3
        - OpenTelemetry.Exporter.Zipkin 1.1.0-beta1
    - Zipkin_gRPCService
        - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
        - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
        - OpenTelemetry.Exporter.Zipkin 1.1.0-beta1

4. Zipkin 環境

    ```bash
    docker run -d -p 9411:9411 openzipkin/zipkin-slim:2
    ```

## 設定方式

- GrpcService

    > 用來儲放共用的 proto 檔及產生的 grpc 實作 (proto 檔的處理不是固定做法)

    1. 安裝 NuGet 套件

        - Google.Protobuf 3.15.7
        - Grpc.Tools 2.36.4
        - Grpc.Core 2.36.4

    2. 設定 .ptoto build action

        > 將 .proto 產生 service 的對象設為 `Both`

        ```xml
        <ItemGroup>
            <Protobuf Include="Protos\greet.proto" GrpcServices="Both" />
        </ItemGroup>
        ```

- Zipkin_gRPCClient

    > 這邊以 console 專案 (ASP.NET Core 的建立方式以 Zipkin_gRPCService 相似) 來建立 gRPC client

    1. 安裝 NuGet 套件

        - Grpc.Net.Client 2.36.0
        - Microsoft.Extensions.Configuration.Json 5.0.0
        - OpenTelemetry.Instrumentation.GrpcNetClient 1.0.0-rc3
        - OpenTelemetry.Exporter.Zipkin 1.1.0-beta1

    2. 加入 Zipkin 設定值

        ```json
        {
            "Zipkin": {
                "ServiceName": "zipkin-grpc-client",
                "Endpoint": "http://localhost:9411/api/v2/spans"
            }
        }
        ```

    3. 註冊 Zipkin

        ```cs
        using var tracerProvider = Sdk.CreateTracerProviderBuilder()
                .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(config.GetValue<string>("Zipkin:ServiceName")))
                .AddGrpcClientInstrumentation()
                .AddZipkinExporter(zipkinOptions =>
                {
                    zipkinOptions.Endpoint = new Uri(config.GetValue<string>("Zipkin:Endpoint"));
                })
                .Build(); 
        ```

- Zipkin_gRPCService

    1. 安裝 NuGet 套件

        - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
        - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
        - OpenTelemetry.Exporter.Zipkin 1.1.0-beta1

    2. 加入 Zipkin 設定值

        ```json
        {
            "Zipkin": {
                "ServiceName": "zipkin-grpc-server",
                "Endpoint": "http://localhost:9411/api/v2/spans"
            }
        }
        ```

    3. 註冊 Zipkin

        ```cs
        services.AddOpenTelemetryTracing((builder) => builder
                .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(this.Configuration.GetValue<string>("Zipkin:ServiceName")))
                .AddAspNetCoreInstrumentation()
                .AddZipkinExporter(zipkinOptions =>
                {
                    zipkinOptions.Endpoint = new Uri(Configuration.GetValue<string>("Zipkin:Endpoint"));
                }));
        ```

- 實際效果

    ![1result](https://user-images.githubusercontent.com/3851540/113512327-b50b9400-9596-11eb-8899-546e347b4e02.png)

    ![2result](https://user-images.githubusercontent.com/3851540/113512333-b76dee00-9596-11eb-8eb3-1c559c45c228.png)

## 心得

整體使用下來，設定方式與 [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](aspdotnet-core-opentelemetry-grpc-jaeger) 幾乎一樣，但是整個 tracing 的 span 會有些延遲，讓我以為是 bug，另外是也會出現看似非預期的 span 資料，但實際 span 又沒有該資料

![3issue](https://user-images.githubusercontent.com/3851540/113512335-b89f1b00-9596-11eb-8ccb-95fbaaa5114d.png)

完整程式碼請參考：[yowko/aspdotnet-core-opentelemetry-grpc](https://github.com/yowko/aspdotnet-core-opentelemetry-grpc)

## 參考資訊

1. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger)
2. [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
3. [OpenTelemetry .NET reaches v1.0](https://devblogs.microsoft.com/dotnet/opentelemetry-net-reaches-v1-0/?WT.mc_id=DOP-MVP-5002594)
4. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
5. [open-telemetry/opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/tree/main/examples/AspNetCore)
6. [yowko/aspdotnet-core-opentelemetry-grpc](https://github.com/yowko/aspdotnet-core-opentelemetry-grpc)
