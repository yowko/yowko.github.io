---
title: "使用 Elasticdump 來匯入與匯出 Elasticsearch index"
date: 2021-04-02T21:30:00+08:00
lastmod: 2021-07-30T21:30:31+08:00
draft: false
tags: ["EFK","Tools"]
slug: "elasticdump"
---

## 使用 Elasticdump 來匯入與匯出 Elasticsearch index

外網的測試環境因為有外部服務介接的關係，有較多機會出現 exception，為了方便 debug，所以想要將外網的 log 倒回內網

之前確實也真的將原始 log 搬回內網，但這麼一來不僅需要 sync log files 還需要額外的 fluentd 來 parse log 後再重新打入 Elasticsearch

所有動作依照外網環境再執行一次，整個過程不是有錯，但總覺得多耗費了一次工，所以興起將處理過後的 elasticsearch index sync 回內網的想法，快速紀錄一下做法

## 基本環境說明

1. macOS Big Sur 11.2.3
2. docker desktop 3.2.2 (61853)

    - fluent/fluentd:v1.12.0-debian-1.0
    - yowko/fluentd-elasticsearch:1.0.0
    - kibana:7.6.2
    - elasticsearch:7.6.2

3. node 15.12.0
4. npm 7.6.3
5. elasticdump 6.66.1s

## 使用方式

1. 安裝 elasticdump

    ```bash
    npm install elasticdump -g
    ```

2. 匯出 Elasticsearch index

    > 這邊以匯出為 json 檔案為例，可以直接匯入 Elasticsearch 或是存成 gzip

    - 語法

        > 這邊 `--input` 指的是想要匯出資料的 Elasticsearch url，`--output` 則是檔案

        ```bash
        elasticdump \
        --input={protocol}://[{username}:{password}@]{host}:{port}/{index} \
        --output={FilePath} \
        --type=data
        ```

    - 範例

        ```bash
        elasticdump \
        --input=http://admin:pass.123@localhost:9200/yowkotest.log \
        --output=yowkotest.json \
        --type=data \
        ```

3. 匯入 Elasticsearch index

    > 這邊我沒看到官網文件提到可以直接使用 gzip

    - 語法

        > 這邊 `--input` 指的是想要匯入資料的檔案，`--output` 則是匯入資料的目標 Elasticsearch url

        ```bash
        elasticdump \
        --input={FilePath} \
        --output={protocol}://[{username}:{password}@]{host}:{port}/{index}
        ```

    - 範例

        ```bash
        elasticdump \
        --input=yowkotest.json \
        --output=http://admin:pass.123@localhost:9200/fromimport
        ```

## 心得

個人覺得 elasticdump 設計的滿好用的，匯出與匯入都可以使用相同參數，在不同情境下參數可以有不同的意義，相當彈性

poc 時測試下來覺得 batch size (`--limit` 預設是 `100`) 與 request 間隔 (`--concurrencyInterval` 預設是 `5000` 毫秒) 需要調整，以預設值測試的結果：約 13mb 的 index 需要 20 分鐘才能匯出，也需要差不多 20 分鐘才能匯入；我嘗試調整成 `--limit=10000` 與 `--concurrencyInterval=1000` 約在 3 分鐘，不過實際的時間與資料的狀況有關，使用前建議要重新調整參數與評估耗時，這邊只是提醒有機會縮短匯出匯入的時間

## 參考資訊

1. [elasticsearch-dump/elasticsearch-dump](https://github.com/elasticsearch-dump/elasticsearch-dump)
