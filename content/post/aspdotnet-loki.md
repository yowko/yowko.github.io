---
title: "將 ASP.NET Core 透過 Grafana Loki 來顯示"
date: 2023-09-04T00:30:00+08:00
lastmod: 2023-09-04T00:30:31+08:00
draft: false
tags: ["aspdotnet","loki","grafana"]
slug: "aspdotnet-loki"
---

## 將 ASP.NET Core 透過 Grafana Loki 來顯示

之前筆記 [使用 Docker Compose 啟動 Grafana Loki](/docker-compose-grafana-loki) 提到過去幾年時間都是透過 Elastic Stack 來處理 log 集中化，因為團隊正在評估下一代產品所使用的 technical stack 於是將 Grafana Loki 納入評估。另一則筆記 [Grafana Loki 搭配 Fluent Bit"](/grafana-loki-fluentbit) 紀錄到如何使用過去的 fluent-bit 來 parsing log file，以降低從 EFK 轉換至 Grafana Loki 的使用門檻，加上團隊還是以 log file 使用為鋌，但站在學術研究的立場，今天來看看如何將 ASP.NET Core 的 log 透過 Loki 顯示在 Grafana 上

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 0.16.1(15815)
3. container images
    - grafana/loki:2.8.0
    - grafana/promtail:2.8.0
    - grafana/grafana:10.0.3
4. NuGet library

    - Serilog.Sinks.Grafana.Loki 8.1.0
    - Serilog.Extensions.Hosting 7.0.0
    - Serilog.AspNetCore 7.0.0
    - Serilog.Sinks.Console 4.1.0

5. docker-compose.yaml

    ```yaml
    version: "3"

    networks:
      loki:

    services:
      loki:
        image: grafana/loki:2.6.0
        ports:
          - "3100:3100"
        command: -config.file=/etc/loki/local-config.yaml
        networks:
          - loki
      promtail:
        image: grafana/promtail:2.6.0
        volumes:
          - /var/log:/var/log
        command: -config.file=/etc/promtail/config.yml
        networks:
          - loki
      grafana:
        image: grafana/grafana:latest
        ports:
          - "3000:3000"
        networks:
          - loki
    ```

## 設定方式

1. 安裝 NuGet 套件

    - Serilog.Sinks.Grafana.Loki

        ```bash
        dotnet add package Serilog.Sinks.Grafana.Loki --version 8.1.0
        ```

    - Serilog.Sinks.Console

        ```bash
        dotnet add package Serilog.Sinks.Console --version 4.1.0
        ```

    - Serilog.Extensions.Hosting

        ```bash
        dotnet remove package Serilog.Extensions.Hosting --version 7.0.0 
        ```

2. 準備 `Serilog` 設定 (非必要，可以使用 serilog api 設定)

    > - `Serilog.WriteTo.Args.labels`：是用來設定固定的 Loki Label，這邊以 key 為 `app`，value 為 `web_app_GrafanaLoki`
    > - `Serilog.WriteTo.Args.propertiesAsLabels`：指定 log 的 property 名稱以 Loki Label 處理 (預設有 `level`，就是 log level)，這邊新增 `from` 用來做為後續動態給值用

    {{< gist yowko 86ce56fd8dd42af16d12ca5b308c18b7 >}}

3. 設定 Log

    - Serilog static Log

        - 透過 serilog api 設定

            {{< gist yowko e96d37c6d5d3ba010ea603f3f40816c7 >}}

        - 透過 appsettings.json 設定

            {{< gist yowko e3b479bd72e5aa4966c167679993c7b5 >}}

    - ILogger

        - 透過 serilog api 設定

            {{< gist yowko 1e88cf3d9a4760908ef4f83516085035 >}}

        - 透過 appsettings.json 設定

            ```cs
            builder.Host.UseSerilog((context, configuration) => { configuration.ReadFrom.Configuration(context.Configuration); });
            ```

4. 使用自訂的 Loki Label

    搭配前面提到的特性：`Serilog.WriteTo.Args.propertiesAsLabels` 可以將特定的 log property 以 Loki Label 方式處理 (以 `from` 為例)，在寫入 log 指定 `from` property 的 value 即可

    ```cs
    using var logCtx = LogContext.PushProperty("from", "GrafanaLoki2");
    ```

    - 使用 Serilog static Log

        ```cs
        Log.Warning("Serilog Warning with Property from");
        ```

    - 使用 ILogger

        ```cs
        _logger.LogInformation("ILogger Information with Property from");
        ```

5. 實際效果

    ![1result](https://github.com/yowko/picsbed/assets/3851540/2572d84c-008b-4181-a6c3-543c78b05c4f)

## 心得

雖然可以使用 Serilog static Log 與 ILogger 兩者方式，但我個人比較偏好 ILogger，讓所以 logger 都使用同樣的做法也可以統一所有 log 內容，避免有的 log 是 ILogger 寫出來的就沒 output 至某個 sink，會一併紀錄起來，純粹只是因為網路上的範例還有不少使用 Serilog static Log，甚至有些官方範例也是這麼用的

Serilog.Sinks.Grafana.Loki 沒有支援 `outputTemplate` 設定，另外就是文件稍嫌不足，資料不是很好找，但可以成功把資料送到 Loki 上，我應該要知足了XD

完整程式碼可以參考 [yowko/aspnetcorgrafanaloki](https://github.com/yowko/aspnetcorgrafanaloki)

## 參考資訊

1. [使用 Docker Compose 啟動 Grafana Loki](/docker-compose-grafana-loki)
2. [Grafana Loki 搭配 Fluent Bit"](/grafana-loki-fluentbit)
3. [serilog-contrib/serilog-sinks-grafana-loki](https://github.com/serilog-contrib/serilog-sinks-grafana-loki?WT.mc_id=DOP-MVP-5002594)
4. [serilog/serilog-settings-configuration/sample/Sample/appsettings.json](https://github.com/serilog/serilog-settings-configuration/blob/dev/sample/Sample/appsettings.json)
5. [serilog/serilog-aspnetcore](https://github.com/serilog/serilog-aspnetcore)
6. [serilog-contrib/serilog-sinks-grafana-loki](https://github.com/serilog-contrib/serilog-sinks-grafana-loki/)
7. [serilog-contrib/serilog-sinks-grafana-loki/Application settings](https://github.com/serilog-contrib/serilog-sinks-grafana-loki/wiki/Application-settings)
8. [Message Templates](https://messagetemplates.org/)
9. [yowko/aspnetcorgrafanaloki](https://github.com/yowko/aspnetcorgrafanaloki)
