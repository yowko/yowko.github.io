---
title: "將 ASP.NET 的 Trace 整合至 Grafana Tempo"
date: 2023-11-29T00:30:00+08:00
lastmod: 2023-11-29T00:30:31+08:00
draft: false
tags: ["docker","grafana","tempo","aspdotnet","csharp"]
slug: "aspdotnet-tempo"
---

## 將 ASP.NET 的 Trace 整合至 Grafana Tempo

之前筆記 [使用 Docker Compose 啟動 Grafana Tempo](/docker-compose-grafana-tempo/) 紀錄到如何透過 docker compose 快速建立 Grafana Tempo 測試環境，今天來看看如何將 ASP.NET 的 Trace 整合至 Grafana Tempo

過去 trace 相關筆記如下：

- OpenTracing
    - [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
    - [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
    - [使用 Jaeger 追蹤 ASP.NET Core 中的 class 呼叫](/jaeger-trace-aspdotnet-core-class-call/)
- OpenTelemetry
    - [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger/)
    - [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-zipkin/)
    - [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-jaeger/)
    - [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-zipkin/)

## 基本環境說明

1. macOS Sonoma 14.1.1 (Apple M2 Pro)
2. OrbStack Version 1.1.0 (16370)
3. container images
    - grafana/tempo:r124-c00e7ef
    - grafana/grafana:10.2.2
    - prom/prometheus:v2.48.0
4. Grafana Tempo 環境建立請參考之前筆記 [使用 Docker Compose 啟動 Grafana Tempo](/docker-compose-grafana-tempo/)
5. .NET SDK 8.0.100
6. NuGet Library

    > 這邊 NuGet Library 的原則是有正式版就用正式版，像是 `OpenTelemetry.Instrumentation.AspNetCore` 跟 `OpenTelemetry.Instrumentation.Http` 沒有發行過正式版就改用 beta 版本

    - OpenTelemetry.Exporter.Console 1.6.0
    - OpenTelemetry.Exporter.OpenTelemetryProtocol 1.6.0
    - OpenTelemetry.Extensions.Hosting 1.6.0
    - OpenTelemetry.Instrumentation.AspNetCore 1.6.0-beta.3
    - OpenTelemetry.Instrumentation.Http 1.6.0-beta.3

## 設定方式

情境說明：

- OtlpClient (Web Api 專案) 中的 `/getweatherforecast` endpoint，加上 activity (span)
- 使用 OtlpClient 本身的 `GetWeatherAsync` method，加上 activity 並加入 tag
- 呼叫 OtlpServer (Web Api 專案) 的 `/weatherforecast` endpoint

1. 安裝 NuGet Library

    - OpenTelemetry.Exporter.Console 1.6.0

        > 這個主要是用來在 console 上顯示 trace 資訊，方便除錯

    - OpenTelemetry.Exporter.OpenTelemetryProtocol 1.6.0
    - OpenTelemetry.Extensions.Hosting 1.6.0
    - OpenTelemetry.Instrumentation.AspNetCore 1.6.0-beta.3
    - OpenTelemetry.Instrumentation.Http 1.6.0-beta.3

2. OtlpServer:Program.cs

    {{< gist yowko 699df06242b8db3d5531f9dc51326117 "OtlpServer-Program.cs" >}}

3. OtlpClient:Program.cs

    {{< gist yowko 699df06242b8db3d5531f9dc51326117 "OtlpClient-Program.cs" >}}

    [tracing.AddSource(sSource.Name );](https://gist.github.com/yowko/699df06242b8db3d5531f9dc51326117#file-otlpclient-program-cs-L31) 很重要，是為後續的 acitvity (span) 設定 parent source，如果沒有這行，後續的 activity 都長不出來

## 心得

1. 實際效果

    - Home --> Explore

        ![1explorer](https://github.com/yowko/picsbed/assets/3851540/2fb698ad-bab3-4f05-99e6-22a1decbdb51)

    - Outline 選 `Tempo` & Query tpye 選 `Search`

        > 列出 trace 內容

        ![2tracesource](https://github.com/yowko/picsbed/assets/3851540/dec2a31d-253c-4b01-945c-564e5519adad)

        ![3trace](https://github.com/yowko/picsbed/assets/3851540/c91f0eb8-d78b-49bb-95ac-02c1ffac5605)

    - 點選 `Trace ID` 後，可以展開該 trace id 下的 activity (span)

        ![1result](https://github.com/yowko/picsbed/assets/3851540/678d00fa-0d0e-4b8a-bcc8-aab1e01c48a5)

        ![2tag](https://github.com/yowko/picsbed/assets/3851540/4ed887c0-5be1-4a4c-b2e0-65f3fd675868)

2. 沒有設定 [tracing.AddSource(sSource.Name );](https://gist.github.com/yowko/699df06242b8db3d5531f9dc51326117#file-otlpclient-program-cs-L31)

    ![3nosource](https://github.com/yowko/picsbed/assets/3851540/069da725-897e-42fc-9574-3ab620b0d6eb)

3. OpenTelemetry 的 Github 上有 .NET 的範例：[Github:open-telemetry/opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/examples/AspNetCore/Program.cs) 但寫法上與 Microsoft 官網：[Microsoft Learn:.NET observability with OpenTelemetry](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel?WT.mc_id=DOP-MVP-5002594) 不同，可以多參考看不同寫法

完整程式碼請參考：[Github:yowko/OtlpWebApi](https://github.com/yowko/OtlpWebApi)

## 參考資料

1. [使用 Docker Compose 啟動 Grafana Tempo](/docker-compose-grafana-tempo/)
2. [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
3. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
4. [使用 Jaeger 追蹤 ASP.NET Core 中的 class 呼叫](/jaeger-trace-aspdotnet-core-class-call/)
5. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger/)
6. [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-zipkin/)
7. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-jaeger/)
8. [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-zipkin/)
9. [Github:open-telemetry/opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/examples/AspNetCore/Program.cs)
10. [Microsoft Learn:.NET observability with OpenTelemetry](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel?WT.mc_id=DOP-MVP-5002594)
11. [Microsoft Learn:Adding distributed tracing instrumentation](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/distributed-tracing-instrumentation-walkthroughs?WT.mc_id=DOP-MVP-5002594)
12. [Microsoft Learn:.NET distributed tracing concepts](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/distributed-tracing-concepts?WT.mc_id=DOP-MVP-5002594)
13. [Github:yowko/OtlpWebApi](https://github.com/yowko/OtlpWebApi)
