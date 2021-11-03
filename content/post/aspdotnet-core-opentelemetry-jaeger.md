---
title: "[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core"
date: 2021-04-03T21:30:00+08:00
lastmod: 2021-04-03T21:30:31+08:00
draft: false
tags: ["Jaeger","OpenTelemetry","ASP.NET Core"]
slug: "aspdotnet-core-opentelemetry-jaeger"
---

## [Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core

前幾天看到微軟官方部落格 [OpenTelemetry .NET reaches v1.0](https://devblogs.microsoft.com/dotnet/opentelemetry-net-reaches-v1-0/?WT.mc_id=DOP-MVP-5002594) 公開了 OpenTelemetry .NET v1.0 版，之前在研究 OpenTracing 也有考慮過 OpenTelemetry 畢竟大一統的規格未來性跟前景應該都比較好，但當時相關套件在 .NET 環境中還相當欠缺；既然看到官方公開宣告了，想必已達可用水準，立馬來測試看看吧

如果想與使用 OpenTracing 做比較可以參考之前筆記 [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)

## 基本環境說明

1. macOS Big Sur 11.2.3
2. docker desktop 3.2.2 (61853)

    - jaegertracing/all-in-one:1.22

3. .NET Core SDK 5.0.103

    - ASP.NET Core 預設專案範本

4. NuGet packages

    - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
    - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
    - OpenTelemetry.Exporter.Jaeger 1.1.0-beta1

## 設定方式

1. 建立 Jaeger 環境

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

2. 安裝 NuGet 套件

    - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
    - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
    - OpenTelemetry.Exporter.Jaeger 1.1.0-beta1

3. 新增 config

    ```json
    {
        "Jaeger": {
          "ServiceName": "jaeger-test",
          "AgentHost": "localhost",
          "AgentPort": 6831
        }
    }
    ```

4. 註冊 Jaeger

    > Startup.cs 的 `ConfigureServices` 方法

    ```cs
     services.AddOpenTelemetryTracing((builder) => builder
        .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(this.Configuration.GetValue<string>("Jaeger:ServiceName")))
        .AddAspNetCoreInstrumentation()
        .AddJaegerExporter());

        services.Configure<JaegerExporterOptions>(this.Configuration.GetSection("Jaeger"));
    ```

5. 實際效果

    ![1jaegerui1](https://user-images.githubusercontent.com/3851540/112724785-ea3d3400-8f4f-11eb-9b58-883b16070eb2.png)

    ![2jaeger2](https://user-images.githubusercontent.com/3851540/112724787-ed382480-8f4f-11eb-9a15-375d0aa3103f.png)

## 心得

整體設定流程與 OpenTracing 類似，但整合度較高 (註冊時有較完整的 extention method)，使用上算便利

但看得出相關專案應該還是調整中，不少相對應的 NuGet 套件需要使用 beta 版本才能安裝使用

完整程式碼可以參考：[yowko/aspdotnet-core-opentelemetry](https://github.com/yowko/aspdotnet-core-opentelemetry)

## 參考資訊

1. [OpenTelemetry .NET reaches v1.0](https://devblogs.microsoft.com/dotnet/opentelemetry-net-reaches-v1-0/?WT.mc_id=DOP-MVP-5002594)
2. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
3. [open-telemetry/opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/tree/main/examples/AspNetCore)
4. [yowko/aspdotnet-core-opentelemetry](https://github.com/yowko/aspdotnet-core-opentelemetry)
