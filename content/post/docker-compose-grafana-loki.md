---
title: "使用 Docker Compose 啟動 Grafana Loki"
date: 2023-08-08T00:30:00+08:00
lastmod: 2023-08-08T00:30:31+08:00
draft: false
tags: ["docker","grafana","loki"]
slug: "docker-compose-grafana-loki"
---

## 使用 Docker Compose 啟動 Grafana Loki

過去幾年時間都是透過 Elastic Stack 來處理 log 集中化，從一開始使用 ELK (Elasticsearch, Logstash , Kibana) 到後來使用 EFK (Elasticsearch , Fluentd , Kibana) 到目前的 EFK (Elasticsearch, Fluent Bit , Kibana)，其中改變的部份是 log 處理工具：Logstash --> Fluentd --> Fluent Bit，這個改變主要是工具處理的效率與資源的耗用量

最近正在評估團隊下一代產品的 technical stack，所以將 Loki 納入評估的一部份，其中包含三個組件：

- Promtail 負責 log 蒐集並將 log 發送給 Loki，對應 Logstash、fluentd、fluent-bit
- Loki 負責 log 儲存與處理查詢，對應 Elasticsearch
- Grafana 用來查詢與顯示 log，對應 Kibana

## 基本環境說明

1. macOS Ventura 13.4
2. OrbStack 0.13.0(1910)
3. container images
    - grafana/loki:2.8.0
    - grafana/promtail:2.8.0
    - grafana/grafana:10.0.3

## 設定方式

1. Loki config：`loki-config.yaml`

    {{< gist yowko 151c473bdeb0d56bc850c05471bab1aa >}}

2. Promtail config：`promtail-local-config.yml`

    {{< gist yowko b6e1c71fd3b8588f49d1c84e4a15dc59 >}}

3. docker-compose：`docker-compose.yml`

    {{< gist yowko 5504c5040aa93c8e275f94e116fa041a >}}

## 實際使用

我還不知道有沒有其他更便利的方式來查詢 log，先紀錄一下知道的用法供參考

1. Home --> Explore

    ![1step1](https://github.com/yowko/picsbed/assets/3851540/59255ad5-c73e-42c1-9894-811302522e0f)

2. 選擇 `Label browser`

    > 如果知道 label filter 該下什麼條件，當然也可以直接輸入

    ![2labelbrowser](https://github.com/yowko/picsbed/assets/3851540/c473b2b7-804b-4fb9-a44c-2c1813d13338)

3. 選擇 `filename` 或是 `job` 的 label 當做條件

    ![3labelfilter](https://github.com/yowko/picsbed/assets/3851540/6638b65e-6ff2-48c9-8223-c1dc97ac6323)

4. 實際效果

    ![4result](https://github.com/yowko/picsbed/assets/3851540/8c2ad040-6451-45d5-b6ea-c230e9bd16da)

    > 1. 時間序列的 log 數量統計
    > 2. 每筆 log 內容

## 心得

1. 預設呈現 log 時間以 paring 的時間非 log 實際發生時間
2. Promtail 在蒐集 log 時沒有做欄位或是型別處理
3. Loki 不像是 Elasticsearch 會針對內容做全文檢索
4. 官網上的 Loki dashboard 都要求同時設定 Loki 與 Prometheus

## 參考資訊

1. [Getting started](https://grafana.com/docs/loki/latest/getting-started/)
2. [Install Grafana Loki with Docker or Docker Compose](https://grafana.com/docs/loki/latest/installation/docker/)
3. [日誌系統 Loki - Promtail 詳解](https://www.readfog.com/a/1642167415174434816)
