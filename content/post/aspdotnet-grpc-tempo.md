---
title: "將 ASP.NET gRPC 的 Trace 整合至 Grafana Tempo"
date: 2023-12-01T00:30:00+08:00
lastmod: 2023-12-01T00:30:31+08:00
draft: false
tags: ["docker","grafana","tempo","aspdotnet","csharp","grpc"]
slug: "aspdotnet-grpc-tempo"
---

## 將 ASP.NET gRPC 的 Trace 整合至 Grafana Tempo

之前筆記 [將 ASP.NET 的 Trace 整合至 Grafana Tempo](/aspdotnet-tempo/) 紀錄到如何將 ASP.NET 的 Trace 整合至 Grafana Tempo，今天來看看如何將 ASP.NET gRPC 的 Trace 整合至 Grafana Tempo

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
    - OpenTelemetry.Instrumentation.GrpcNetClient 1.6.0-beta.3
    - Grpc.AspNetCore 2.59.0
    - Grpc.Net.Client 2.59.0

## 設定方式

情境說明：

- OtlpGrpcClient (Web Api 專案) 中的 `/getweatherforecastgrpc` endpoint，加上 activity (span)
- 使用 OtlpGrpcClient 本身的 `GetWeatherGrpcAsync` method，加上 activity 並加入 tag
- 呼叫 OtlpGrpcServer (Grpc Service 專案) 的 `Greeter/SayHello` endpoint

1. 安裝 NuGet Library

    - OpenTelemetry.Exporter.Console 1.6.0

        > 這個主要是用來在 console 上顯示 trace 資訊，方便除錯

    - OpenTelemetry.Exporter.OpenTelemetryProtocol 1.6.0
    - OpenTelemetry.Extensions.Hosting 1.6.0
    - OpenTelemetry.Instrumentation.AspNetCore 1.6.0-beta.3
    - OpenTelemetry.Instrumentation.Http 1.6.0-beta.3
    - OpenTelemetry.Instrumentation.GrpcNetClient 1.6.0-beta.3

        > 僅 OtlpGrpcClient 需要

    - Grpc.AspNetCore 2.59.0

        > 僅 OtlpGrpcClient 需要

    - Grpc.Net.Client 2.59.0

        > 僅 OtlpGrpcClient 需要

2. OtlpGrpcServer:Program.cs

    {{< gist yowko 58a0b75afced949044399cff39227cde "OtlpGrpcServer_Program.cs" >}}

3. OtlpGrpcClient:Program.cs

    {{< gist yowko 58a0b75afced949044399cff39227cde "OtlpGrpcClient_Program.cs" >}}

## 心得

1. 實際效果

    - Home --> Explore

        ![1explorer](https://github.com/yowko/picsbed/assets/3851540/2fb698ad-bab3-4f05-99e6-22a1decbdb51)

    - Outline 選 `Tempo` & Query tpye 選 `Search`

        > 列出 trace 內容

        ![2tracesource](https://github.com/yowko/picsbed/assets/3851540/dec2a31d-253c-4b01-945c-564e5519adad)

        ![3trace](https://github.com/yowko/picsbed/assets/3851540/c91f0eb8-d78b-49bb-95ac-02c1ffac5605)

    - 點選 `Trace ID` 後，可以展開該 trace id 下的 activity (span)

        ![1trace](https://github.com/yowko/picsbed/assets/3851540/1b28b6a8-b3db-4c93-ae95-080e5d6987bd)

        ![2tagok](https://github.com/yowko/picsbed/assets/3851540/ba4464b0-4952-4828-aa07-f8387f0a2b60)

2. OtlpGrpcClient 有沒有 [`tracing.AddHttpClientInstrumentation();`](https://gist.github.com/yowko/58a0b75afced949044399cff39227cde#file-otlpgrpcclient_program-cs-L25) 差很多

    - 有加入 `tracing.AddHttpClientInstrumentation();` 的結果

        ![3withhttp](https://github.com/yowko/picsbed/assets/3851540/7ae42d70-1381-4791-99f3-b1120bc2e12c)

    - 沒有加入 `tracing.AddHttpClientInstrumentation();` 的結果

        ![4withouthttp](https://github.com/yowko/picsbed/assets/3851540/7389961a-ee43-44a0-b394-cb7bf785be2f)

3. 使用上與 ASP.NET WebApi：[將 ASP.NET 的 Trace 整合至 Grafana Tempo](/aspdotnet-tempo/) 幾乎相同，只是 OtlpGrpcClient 多了一個 [`GrpcInstrumentation` 設定：`tracing.AddGrpcClientInstrumentation();`](https://gist.github.com/yowko/58a0b75afced949044399cff39227cde#file-otlpgrpcclient_program-cs-L26)

4. ASP.NET 8 在 macOS 上啟動 gRPC Service 時不用在刻意使用 Http1 與 insecure gRPC 了，之前筆記 [在 macOS 上啟動 gRPC Service 時 HTTP/2 的問題](/grpc-service-http2-macos/) 有提到這個問題，現在 ASP.NET 8 終於解決  HTTP/2 的問題了，詳情請看 官方文件 [Microsoft Learn:HTTP/2 over TLS (HTTPS) support on macOS in Kestrel](https://learn.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-8.0?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594#http2-over-tls-https-support-on-macos-in-kestrel)

完整程式碼請參考：[Github:yowko/otlpgrpc](https://github.com/yowko/otlpgrpc)

## 參考資料

1. [使用 Docker Compose 啟動 Grafana Tempo](/docker-compose-grafana-tempo/)
2. [將 ASP.NET 的 Trace 整合至 Grafana Tempo](/aspdotnet-tempo/)
3. [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
4. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
5. [使用 Jaeger 追蹤 ASP.NET Core 中的 class 呼叫](/jaeger-trace-aspdotnet-core-class-call/)
6. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger/)
7. [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-zipkin/)
8. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-jaeger/)
9. [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-zipkin/)
10. [在 macOS 上啟動 gRPC Service 時 HTTP/2 的問題](/grpc-service-http2-macos/)
11. [Microsoft Learn:HTTP/2 over TLS (HTTPS) support on macOS in Kestrel](https://learn.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-8.0?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594#http2-over-tls-https-support-on-macos-in-kestrel)
12. [Github:yowko/otlpgrpc](https://github.com/yowko/otlpgrpc)
