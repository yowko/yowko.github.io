---
title: "使用 Docker Compose 啟動 Grafana Tempo"
date: 2023-11-28T00:30:00+08:00
lastmod: 2023-11-28T00:30:31+08:00
draft: false
tags: ["docker","grafana","tempo"]
slug: "docker-compose-grafana-tempo"
---

## 使用 Docker Compose 啟動 Grafana Tempo

過去筆記紀錄到團隊由 OpenTracing 轉換到 OpenTelemetry，其中一個原因是 OpenTracing 的專案已經不再維護，另一個原因是 OpenTelemetry 有提供更多的功能，其中一個就是 OpenTelemetry 有提供一個分散式追蹤系統：[Tempo](https://grafana.com/oss/tempo/)，過去只有紀錄 Zipkin 與 Jaeger 的使用，今天就來紀錄一下如何使用 Docker Compose 建立 Grafana Tempo 測試環境

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

最近正在評估團隊下一代產品的 technical stack，所以將 Tempo 納入評估的一部份，其中包含三個組件：

- Grafana 用來查詢與顯示 trace 資料
- Tempo 負責 trace 資料的儲存與處理查詢
- Prometheus 用來紀錄 Service Graph 資料

## 基本環境說明

1. macOS Sonoma 14.1.1 (Apple M2 Pro)
2. OrbStack Version 1.1.0 (16370)
3. container images
    - grafana/tempo:r124-c00e7ef
    - grafana/grafana:10.2.2
    - prom/prometheus:v2.48.0

## 設定方式

設定方式是參考 Grafana Tempo 的 [Github 範例](https://github.com/grafana/tempo/blob/main/example/docker-compose/local) 而來的，可以參考原始內容，我個人拿掉了 `k6-tracing` 與 tempo 的 jaeger 跟 zipkin 的相容設定

1. `admin_password` (選擇性)

    這是個人偏好：透過 admin 與 admin_password 來登入 Grafana，所以在 docker-compose.yml 中加上了 admin_password 的設定，如果不想設定 admin 密碼，後面的參數會有些不同會額外說明

    ```yaml
    pass.123
    ```

2. prometheus.yaml

    {{< gist yowko 74bc4e0b001d761165b1a09d3e982eb2 "prometheus.yaml" >}}

3. tempo.yaml

    {{< gist yowko 74bc4e0b001d761165b1a09d3e982eb2 "tempo.yaml" >}}

4. grafana-datasources.yaml

    {{< gist yowko 74bc4e0b001d761165b1a09d3e982eb2 "grafana-datasources.yaml" >}}

5. docker-compose-tempo.yaml

    {{< gist yowko 74bc4e0b001d761165b1a09d3e982eb2 "docker-compose-tempo.yaml" >}}

## 實際使用

trace 資料來源來使用 [Github Grafana Tempo 範例](https://github.com/grafana/tempo/blob/main/example/docker-compose/local) 中的 `k6-tracing`，這個範例是使用 [k6](https://k6.io/) 來模擬一個簡單的 web server，這個範例會產生一些 trace 資料，我們可以透過 Grafana 來查詢這些 trace 資料

1. Home --> Explore

    ![1explorer](https://github.com/yowko/picsbed/assets/3851540/2fb698ad-bab3-4f05-99e6-22a1decbdb51)

2. Outline 選 `Tempo` & Query tpye 選 `Search`

    > 列出 trace 內容

    ![2tracesource](https://github.com/yowko/picsbed/assets/3851540/dec2a31d-253c-4b01-945c-564e5519adad)

    ![3trace](https://github.com/yowko/picsbed/assets/3851540/c91f0eb8-d78b-49bb-95ac-02c1ffac5605)

3. Outline 選 `Tempo` & Query tpye 選 `Service Graph`

    > 列出 trace 的服務關係

    ![4servicegraph](https://github.com/yowko/picsbed/assets/3851540/63b954bb-5f2a-42df-b6d2-d7867c70c752)

    ![4servicegraph](https://github.com/yowko/picsbed/assets/3851540/61a55e19-9db3-4d17-9086-5585dd912a07)

## 心得

官網文件 [Quick start for Tempo](https://grafana.com/docs/tempo/latest/getting-started/docker-example/) 很直覺，原則上照著文件做就可以了，不過有幾點是我做這份筆記的原因：

1. 官網文件 [Quick start for Tempo](https://grafana.com/docs/tempo/latest/getting-started/docker-example/)與官方 [Github Grafana Tempo 範例](https://github.com/grafana/tempo/blob/main/example/docker-compose/local) 比較偏像是 demo 用的，不適合用來進行其他整合應用的基礎環境
2. 官方文件中的 `k6-tracing` 會產生一些 trace 資料，但是沒有提供任何的說明，所以我們只能透過 Grafana 來查詢這些 trace 資料
3. 官方 [Github Grafana Tempo 範例](https://github.com/grafana/tempo/blob/main/example/docker-compose/local) 是多個環境的結構，單純 local 開發，就不需要下載整個 repo ，直接參考上面的設定方式就可以了

完整程式碼請參考：[Github:yowko/docker-compose-grafana-tempo](https://github.com/yowko/docker-compose-grafana-tempo)

## 參考資訊

1. [Tempo](https://grafana.com/oss/tempo/)
2. [Quick start for Tempo](https://grafana.com/docs/tempo/latest/getting-started/docker-example/)
3. [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
4. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
5. [使用 Jaeger 追蹤 ASP.NET Core 中的 class 呼叫](/jaeger-trace-aspdotnet-core-class-call/)
6. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-jaeger/)
7. [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core](/aspdotnet-core-opentelemetry-zipkin/)
8. [[Jaeger] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-jaeger/)
9. [[Zipkin] 使用 OpenTelemetry 來追蹤 ASP.NET Core 上的 gRPC 呼叫](/aspdotnet-core-opentelemetry-grpc-zipkin/)
10. [Github Grafana Tempo 範例](https://github.com/grafana/tempo/blob/main/example/docker-compose/local)
11. [Github:yowko/docker-compose-grafana-tempo](https://github.com/yowko/docker-compose-grafana-tempo)
