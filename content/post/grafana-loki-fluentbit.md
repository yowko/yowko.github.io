---
title: "Grafana Loki 搭配 Fluent Bit"
date: 2023-08-15T00:30:00+08:00
lastmod: 2023-08-15T00:30:31+08:00
draft: false
tags: ["docker","grafana","loki"]
slug: "grafana-loki-fluentbit"
---

## Grafana Loki 搭配 Fluent Bit

之前筆記 [使用 Docker Compose 啟動 Grafana Loki](/docker-compose-grafana-loki) 紀錄如何使用 docker compose 快速啟動 Grafana Loki 環境，其中使用的 log 蒐集器是 Grafana 預設的 Promtail，不過 Promtail 功能上只專注在把 log 打到 Loki 不擔負清洗的工作，所以還是得讓 Fluent Bit 上場

## 基本環境說明

1. macOS Ventura 13.4
2. OrbStack 0.15.0(2284)
3. container images
    - grafana/loki:2.8.0
    - grafana/grafana:10.0.3
    - fluent/fluent-bit:2.1.8
    - fluent/fluent-bit:2.1.8-debug

        > - debug 版包含 shell 跟工具可以除錯

4. 測試 log

    ```log
    2023-08-15 09:00:38.192 [188][ERR][Yowko.DomainService.Test.Application.Utilities.ExceptionInterceptor]UserId:TW001|Name:Yowko|DepartmentId:D01
    2023-08-15 09:02:38.192 [198][ERR][Yowko.DomainService.Test.Application.Utilities.ExceptionInterceptor]UserId:TW002|Name:Tsai|DepartmentId:D01

    ```

## 設定方式

1. Loki config：`loki-config.yaml`

    > 這個內容與之前筆記 [使用 Docker Compose 啟動 Grafana Loki](/docker-compose-grafana-loki) 相同

    {{< gist yowko 151c473bdeb0d56bc850c05471bab1aa >}}

2. Fluent Bit 設定

    - Parser config:

        > parser 相關設定需要額外放在其他檔案，不允許放在 fluent bit 主要 config 檔案中

        ![1parsernotvalid](https://github.com/yowko/picsbed/assets/3851540/6a93370c-a956-4047-b16f-03fadb733d86)

        {{< gist yowko 643b399b8d8250a7d8555270b5daedec >}}

        - 可以先使用 [Rubular](https://rubular.com/) 測試 Fluent Bit 用的 regex 是否正確
        - 如果 log 時區不是 UTC ，需要額外指定 `Time_Offset`

            - 錯誤訊息

                ```txt
                [2023/08/15 01:25:23] [error] [output:loki:loki.1] loki:3100, HTTP status=400 Not retrying.
                entry for stream '{department_id="D01", log_level="ERR", name="Tsai", sender="Yowko.DomainService.Test.Application.Utilities.ExceptionInterceptor", user_id="TW002"}' has timestamp too new: 2023-08-15T09:02:38Z
                ```

            - 錯誤截圖

                ![2timestmaptoonew](https://github.com/yowko/picsbed/assets/3851540/ad6a043b-8b8e-479a-9ca6-0a034c9bb314)

    - Fluent Bit config：

        {{< gist yowko 01c1a67b6a0851e2dcbcfbc808f8731b>}}

3. docker compose：`docker-compose-fluentbit.yml`

    > 大致內容與之前筆記 [使用 Docker Compose 啟動 Grafana Loki](/docker-compose-grafana-loki) 差不別，就是把 Promtail 換成 Fluent Bit

    {{< gist yowko 1816bf34c5eb4e176ee7600642187383 >}}

4. 實際效果

    ![3result](https://github.com/yowko/picsbed/assets/3851540/1c7ed52b-3503-4ab0-ba63-922cecee70f6)

## 心得

GrafanaLabs 官網上的文件 [Fluent Bit](https://grafana.com/docs/loki/latest/clients/fluentbit/) 使用 `grafana/fluent-bit-plugin-loki:latest` 但上次更新時間已是兩年前，除此之外也只有 amd64 的版本， arm 版本需要使用其他 tag，而 [Fluent Bit: Official Manual:Loki](https://docs.fluentbit.io/manual/pipeline/outputs/loki) 可以直接使用官方 images：`fluent/fluent-bit` 不用額外安裝 output 的 plugin

除此之外，GrafanaLabs 官網上的文件 [Fluent Bit](https://grafana.com/docs/loki/latest/clients/fluentbit/) 用的參數也是舊的，新版並不相容

## 參考資訊

1. [Fluent Bit: Official Manual:Loki](https://docs.fluentbit.io/manual/pipeline/outputs/loki)
2. [GrafanaLabs:Fluent Bit](https://grafana.com/docs/loki/latest/clients/fluentbit/)
3. [fluent/fluent-bit](https://hub.docker.com/r/fluent/fluent-bit)
4. [Rubular](https://rubular.com/) 
5. [Log Agent - Fluent Bit Parser元件](https://ithelp.ithome.com.tw/articles/10281051)
