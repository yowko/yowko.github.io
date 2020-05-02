---
title: "Fluentd 指定 Elasticsearch Index 名稱"
date: 2020-04-30T21:30:00+08:00
lastmod: 2020-05-02T21:30:31+08:00
draft: false
tags: ["EFK","Tools"]
slug: "fluentd-elasticsearch-index-name"
---

## Fluentd 指定 Elasticsearch Index 名稱

最近在 debug Fluentd parser 時老是覺得 log 沒進到 Elasticsearch，後來索性把預設的 index 換掉，至少一眼就看得出來 index 有沒有被建立起來 - Fluentd 是不是正常運作中

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.5(43884)
3. docker images

    > 詳細資訊可以參考之前筆記 [快速建立 EFK (Elasticsearch Fluentd Kibana) 環境](https://blog.yowko.com/docker-efk)

    - elasticsearch:7.6.2
    - kibana:7.6.2
    - yowko/fluentd-elasticsearch:1.0.0

## 目前狀況

- 啟用 logstash_format

    ```txt
    logstash_format true
    ```

- Fluentd 預設會使用 `logstash-%Y.%m.%d` 做為 index name 

    > 會覆寫 `index_name` 設定

    ![1logformat]()

## 調整方式

1. 方法一： `logstash_format` 搭配 `logstash_prefix`

    - 設定內容

        ```txt
        <match *.**>
            @type elasticsearch
            host 192.168.50.97
            port 9200
            logstash_format true
        </match>
        ```

    - 實際效果

        ![2logstashprefix]()

2. 方法二： 使用 `index_name`

    - 設定內容

        ```txt
        <match *.**>
            @type elasticsearch
            host 192.168.50.97
            port 9200
            index_name yowkotest
        </match>
        ```

    - 實際效果

        ![3indexname]()

## 心得

雖然只是小小設定，但每次忘記都要重新查資料也滿花時間的，趁著這次機會簡單筆記一下，加深印象，之後忘記要找資料也可以比較快囉

## 參考資訊

1. [快速建立 EFK (Elasticsearch Fluentd Kibana) 環境](https://blog.yowko.com/docker-efk)
2. [elasticsearch](https://docs.fluentd.org/output/elasticsearch#parameters)
