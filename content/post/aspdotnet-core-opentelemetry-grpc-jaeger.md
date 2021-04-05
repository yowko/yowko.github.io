---
title: "[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫"
date: 2021-04-04T21:30:00+08:00
lastmod: 2021-04-04T21:30:31+08:00
draft: false
tags: ["Jaeger","OpenTelemetry","gRPC","ASP.NET Core"]
slug: "aspdotnet-core-opentelemetry-grpc-jaeger"
---

## [Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫

之前筆記 [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger) 紀錄到使用 OpenTelemetry 搭配 Jaeger 來追蹤 ASP.NET Core 也曾在 [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/) 紀錄使用 OpenTracing 來追蹤 gRPC 呼叫，因為目前團隊中主要使用 gRPC 提供服務，gRPC 的相關偵錯佔了相當大的比例，今天就快速紀錄一下如何使用 OpenTelemetry 來 追蹤 ASP.NET Core 上的 gRPC 吧

## 基本環境說明

1. macOS Big Sur 11.2.3
2. docker desktop 3.2.2 (61853)

    - jaegertracing/all-in-one:1.22

3. NuGet packages
    - GrpcService
        - Google.Protobuf 3.15.7
        - Grpc.Tools 2.36.4
        - Grpc.Core 2.36.4
    - Jaeger_gRPCClient
        - Grpc.Net.Client 2.36.0
        - Microsoft.Extensions.Configuration.Json 5.0.0
        - OpenTelemetry.Instrumentation.GrpcNetClient 1.0.0-rc3
        - OpenTelemetry.Exporter.Jaeger 1.1.0-beta1
    - Jaeger_gRPCService
        - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
        - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
        - OpenTelemetry.Exporter.Jaeger 1.1.0-beta1

4. jaeger 環境

    ```bash
    docker run -d --name jaeger \
    -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
    -p 5775:5775/udp \
    -p 6831:6831/udp \
    -p 6832:6832/udp \
    -p 5778:5778 \
    -p 16686:16686 \
    -p 14268:14268 \
    -p 14250:14250 \
    -p 9411:9411 \
    jaegertracing/all-in-one:1.22
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

- Jaeger_gRPCClient

    > 這邊以 console 專案 (ASP.NET Core 的建立方式以 Jaeger_gRPCService 相似) 來建立 gRPC client

    1. 安裝 NuGet 套件

        - Grpc.Net.Client 2.36.0
        - Microsoft.Extensions.Configuration.Json 5.0.0
        - OpenTelemetry.Instrumentation.GrpcNetClient 1.0.0-rc3
        - OpenTelemetry.Exporter.Jaeger 1.1.0-beta1

    2. 加入 jaeger 設定值

        ```json
        {
            "Jaeger": {
                "ServiceName": "jaeger-grpc-client",
                "Host": "localhost",
                "Port": 6831
            }
        }
        ```

    3. 註冊 Jaeger

        ```cs
        using var tracerProvider = Sdk.CreateTracerProviderBuilder()
                .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(config.GetValue<string>("Jaeger:ServiceName")))
                .AddGrpcClientInstrumentation()
                .AddJaegerExporter(jaegerOptions =>
                {
                    jaegerOptions.AgentHost = config.GetValue<string>("Jaeger:Host");
                    jaegerOptions.AgentPort = config.GetValue<int>("Jaeger:Port");
                })
                .Build();
        ```

- Jaeger_gRPCService

    1. 安裝 NuGet 套件

        - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
        - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
        - OpenTelemetry.Exporter.Jaeger 1.1.0-beta1

    2. 加入 jaeger 設定值

        ```json
        {
            "Jaeger": {
                "ServiceName": "jaeger-grpc-server",
                "Host": "localhost",
                "Port": 6831
            }
        }
        ```

    3. 註冊 Jaeger

        ```cs
        services.AddOpenTelemetryTracing((builder) => builder
            .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(this.Configuration.GetValue<string>("Jaeger:ServiceName")))
            .AddAspNetCoreInstrumentation()
            .AddJaegerExporter(jaegerOptions =>
            {
                jaegerOptions.AgentHost = this.Configuration.GetValue<string>("Jaeger:Host");
                jaegerOptions.AgentPort = this.Configuration.GetValue<int>("Jaeger:Port");
            }));
        ```

- 實際效果

    ![1result](https://user-images.githubusercontent.com/3851540/113480430-bf5e5d00-94c6-11eb-8346-827c667b7a7b.png)

## 心得

整體使用下來，跟之前筆記 [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger) 中套用在 ASP.NET Core 上的使用經驗雷同，大致概念與 OpenTracing 差不多，但設定的便利性上有不少的提升，我相信再過段時間絕對會取代過去 OpenTracing 的方式成為主流；倒不是現在直接使用 OpenTelemetry 有什麼問題，只不過 OpenTelemetry 多項 NuGet package 仍在 pre-release 階段，正式專案還是不敢直接使用

完整程式碼請參考：[yowko/aspdotnet-core-opentelemetry-grpc](https://github.com/yowko/aspdotnet-core-opentelemetry-grpc)

## 參考資訊

1. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger)
2. [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
3. [OpenTelemetry .NET reaches v1.0](https://devblogs.microsoft.com/dotnet/opentelemetry-net-reaches-v1-0/?WT.mc_id=DOP-MVP-5002594)
4. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](https://blog.yowko.com/jaeger-trace-aspdotnet-core/)
5. [open-telemetry/opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/tree/main/examples/AspNetCore)
6. [yowko/aspdotnet-core-opentelemetry-grpc](https://github.com/yowko/aspdotnet-core-opentelemetry-grpc)
