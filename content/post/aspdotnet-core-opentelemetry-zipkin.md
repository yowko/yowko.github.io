---
title: "[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core"
date: 2021-04-03T22:00:00+08:00
lastmod: 2021-11-03T22:00:31+08:00
draft: false
tags: ["Zipkin","OpenTelemetry","ASP.NET Core"]
slug: "aspdotnet-core-opentelemetry-zipkin"
---

## [Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core

這是延續之前筆記 [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger) 進行嘗試的筆記，過去使用 OpenTracing 時因為 Zipkin 與 .NET Core 的整合程式較低，設定上繁瑣許多，加上無法處理主要想追蹤 gRPC 所以當時並沒有採用，不過目前看相關文件看似可用，就來測試做個紀錄吧

## 基本環境說明

1. macOS Big Sur 11.2.3
2. docker desktop 3.2.2 (61853)

    - openzipkin/zipkin-slim:2

3. .NET Core SDK 5.0.103

    - ASP.NET Core 預設專案範本

4. NuGet packages

    - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
    - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
    - OpenTelemetry.Exporter.Zipkin 1.1.0-beta1

## 設定方式

1. 建立 Jaeger 環境

    ```bash
    docker run -d -p 9411:9411 openzipkin/zipkin-slim:2
    ```

2. 安裝 NuGet 套件

    - OpenTelemetry.Extensions.Hosting 1.0.0-rc3
    - OpenTelemetry.Instrumentation.AspNetCore 1.0.0-rc3
    - OpenTelemetry.Exporter.Zipkin 1.1.0-beta1

3. 新增 config

    ```json
    {
        "Zipkin": {
            "ServiceName": "zipkin-test",
            "Endpoint": "http://localhost:9411/api/v2/spans"
        }
    }
    ```

4. 註冊 Jaeger

    > Startup.cs 的 `ConfigureServices` 方法

    ```cs
    services.AddOpenTelemetryTracing((builder) => builder
            .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(this.Configuration.GetValue<string>("Zipkin:ServiceName")))
            .AddAspNetCoreInstrumentation()
            .AddZipkinExporter());

    services.Configure<ZipkinExporterOptions>(this.Configuration.GetSection("Zipkin"));
    ```

5. 實際效果

    ![1zipkinui1](https://user-images.githubusercontent.com/3851540/112747564-c843c000-8fe8-11eb-9a60-6b91c65042a3.png)

    ![2jaeger2](https://user-images.githubusercontent.com/3851540/112747566-ca0d8380-8fe8-11eb-9812-6c946d60dd74.png)

## 心得

整體設定流程與 [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger) 完全相同，整體使用體驗非常好，跟之前 OpenTracing 完全不同的設定方法差距很大

但有個小問題是設定方式的 [官方 GitHub sample](https://github.com/open-telemetry/opentelemetry-dotnet/blob/5bf2b42379d1d993fc2f1cec1e54ab1aab3bb3f0/examples/AspNetCore/Startup.cs#L71) 有錯，少了 service name 的設定，幸虧與 Jaeger 設定方式完全一樣，沒造成多大的問題

依 [官方 GitHub sample](https://github.com/open-telemetry/opentelemetry-dotnet/blob/5bf2b42379d1d993fc2f1cec1e54ab1aab3bb3f0/examples/AspNetCore/Startup.cs#L71) 程式碼設定會出現 `unknown_service:dotnet`

![3zipkinnoservicename](https://user-images.githubusercontent.com/3851540/112747568-cbd74700-8fe8-11eb-8262-7fe23f59d680.png)

完整程式碼可以參考：[yowko/aspdotnet-core-opentelemetry](https://github.com/yowko/aspdotnet-core-opentelemetry)

## 參考資訊

1. [OpenTelemetry .NET reaches v1.0](https://devblogs.microsoft.com/dotnet/opentelemetry-net-reaches-v1-0/?WT.mc_id=DOP-MVP-5002594)
2. [open-telemetry/opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/tree/main/examples/AspNetCore)
3. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger)
4. [yowko/aspdotnet-core-opentelemetry](https://github.com/yowko/aspdotnet-core-opentelemetry)
