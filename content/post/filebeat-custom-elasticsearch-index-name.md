---
title: "Filebeat 自訂 Elasticsearch index name"
date: 2023-04-10T00:30:00+08:00
lastmod: 2023-04-10T00:30:31+08:00
draft: false
tags: ["elasticsearch"]
slug: "filebeat-custom-elasticsearch-index-name"
---

## Filebeat 自訂 Elasticsearch index name

這個是之前筆記 [使用 filebeat 將 Redis slowlog 存至 Elasticsearch](/redis-slowlog-elasticsearch) 遇到的狀況，筆記使用當時最新版本： `8.6.1`，但團隊在部份環境還在使用 `7.X` 造成 index 名稱雜亂

1. filebeat 7.X 預設 index 名稱是 `filebeat-{agent.version}-{yyyy.MM.dd}-{序號}`，

    ![1filebeat7](https://user-images.githubusercontent.com/3851540/230815034-84e4ef8e-9eeb-4aee-b248-7d4b54a12e39.png)

2. filebeat 8.X 預設 index 名稱是 `filebeat-{agent.version}`

    ![2filebeat8](https://user-images.githubusercontent.com/3851540/230815041-810f0a4c-05be-4a1d-b8b1-110e89f94012.png)

為了符合團隊 kibana dashboard 的處理，調整 index 名稱是必要的，但嘗試了一下發現幾個 google 來的答案沒有提及版本的差異，造成無法設定成期望的名稱，仔細翻閱官方文件才找到原因，紀錄一下加深印象

因為是 [使用 filebeat 將 Redis slowlog 存至 Elasticsearch](/redis-slowlog-elasticsearch) 遇到的狀況，範例主架構就延用上次內容不另外準備，只更新版本

## 基本環境說明

1. macOS Ventura 13.2
2. docker desktop 4.2.0(70708)
    - Engine 20.10.10
    - Compose 1.29.2
3. docker images
    - elastic/elasticsearch:8.7.0
    - elastic/elasticsearch:.7.17.9
    - elastic/kibana:8.7.0
    - elastic/kibana:7.17.9
    - redis:7.0.10
    - docker.elastic.co/beats/filebeat:8.7.0
    - docker.elastic.co/beats/filebeat:7.17.9
4. filebeat config

    ```yaml
    filebeat.modules:
    - module: redis
      log:
        enabled: false
      slowlog:
        enabled: true
        var.hosts: ["redis:6379"]
    output.elasticsearch:
      enabled: true
      hosts: ["http://elasticsearch:9200"]
    ```

5. docker compose

    - elasticsearch 7.17

        ```yaml
        version: "3.9"
        services:
          elasticsearch:
            image: elastic/elasticsearch:7.17.9
            environment:
              - discovery.type=single-node
              - ES_JAVA_OPTS=-Xms1g -Xmx1g
              - xpack.security.enabled=false
            volumes:
              - esdata:/usr/share/elasticsearch/data
            ports:
              - target: 9200
                published: 9200
          kibana:
            image: kibana:7.17.9
            ports:
              - target: 5601
                published: 5601
            depends_on:
              - elasticsearch
          redis:
            image: redis:7.0.10
            ports:
              - target: 6379
                published: 6379
          redisbeat:
            image: docker.elastic.co/beats/filebeat:7.17.9
            volumes:
              - ./redisbeats7.yml:/usr/share/filebeat/filebeat.yml:ro
            depends_on:
              - elasticsearch
        volumes:
          esdata:
            driver: local
        ```

    - elasticsearch 8.7

        ```yaml
        version: "3.9"
        services:
          elasticsearch:
            image: elastic/elasticsearch:8.7.0
            environment:
              - discovery.type=single-node
              - ES_JAVA_OPTS=-Xms1g -Xmx1g
              - xpack.security.enabled=false
            volumes:
              - esdata:/usr/share/elasticsearch/data
            ports:
              - target: 9200
                published: 9200
          kibana:
            image: kibana:8.7.0
            ports:
              - target: 5601
                published: 5601
            depends_on:
              - elasticsearch
          redis:
            image: redis:7.0.10
            ports:
              - target: 6379
                published: 6379
          redisbeat:
            image: docker.elastic.co/beats/filebeat:8.7.0
            volumes:
              - ./redisbeats8.yml:/usr/share/filebeat/filebeat.yml:ro
            depends_on:
              - elasticsearch
        volumes:
          esdata:
            driver: local
        ```

## 設定方式

- filebeat 7:redisbeats7.yml

    - 修改 `setup.ilm.rollover_alias` 與 `setup.ilm.pattern`
    - `setup.ilm.enabled` 預設已啟用

        > 預設 `ilm` 是啟用，index 的 pattern 由 `ilm` 接管，詳細內容可以參考官方文件：[Elastic Docs ›Filebeat Reference [7.17] ›How to guides](https://www.elastic.co/guide/en/beats/filebeat/7.17/change-index-name.html)

    ```yaml
    filebeat.modules:
    - module: redis
      log:
        enabled: false
      slowlog:
        enabled: true
        var.hosts: ["redis:6379"]
    output.elasticsearch:
      enabled: true
      hosts: ["http://elasticsearch:9200"]
    setup.ilm.enabled: auto
    setup.ilm.rollover_alias: "redis"
    setup.ilm.pattern: "{now/d}"
    ```

        ![3redisbeat7](https://user-images.githubusercontent.com/3851540/230815049-dbe09634-5e59-47d1-8a17-4b1e562d26d7.png)

- filebeat 8:redisbeats8.yml

    - 修改 `output.elasticsearch.index`、`setup.template.name` 與 `setup.template.pattern`

        > 修改 `output.elasticsearch.index` 需一併設定 `setup.template.name` 與 `setup.template.pattern`，詳細內容可以參考官方文件：[Elastic Docs ›Filebeat Reference [8.7] ›How to guides](https://www.elastic.co/guide/en/beats/filebeat/8.7/change-index-name.html)

    ```yaml
    filebeat.modules:
    - module: redis
      log:
        enabled: false
      slowlog:
        enabled: true
        var.hosts: ["redis:6379"]
    output.elasticsearch:
      enabled: true
      hosts: ["http://elasticsearch:9200"]
      index: "redis-%{+yyyy.MM}"
    setup.template.name: "redis"
    setup.template.pattern: "redis-*"
    ```

        ![4redisbeat8](https://user-images.githubusercontent.com/3851540/230815053-7e08b4d5-2272-4db3-8df3-70029ac023d1.png)

## 心得

反覆測試了好幾次才發現在 filebeat 不同版本間設定方式不一樣的問題，類似的狀況我還額外在實際部署時遇到：環境使用 helm 安裝 `elastic/filebeat`，filebeat 7 與 8 的 value 有比較大幅度的調整。以我的例子來說，我需要用 8.X 的 chart 但 filebeat 則需要使用 7.X

除此之外我自己測試時的另一個狀況是：filebeat 需要與 elasticsearch 使相同版本，否則會收不到資料，我快速看了 log 是 metadata 不同造成的

完整原始碼：[yowko/filebeat-custom-index](https://github.com/yowko/filebeat-custom-index)

## 參考資訊

1. [使用 filebeat 將 Redis slowlog 存至 Elasticsearch](/redis-slowlog-elasticsearch)
2. [Elastic Docs ›Filebeat Reference [7.17] ›Configure Filebeat](https://www.elastic.co/guide/en/beats/filebeat/7.17/ilm.html)
3. [Elastic Docs ›Filebeat Reference [8.6] ›Configure Filebeat](https://www.elastic.co/guide/en/beats/filebeat/8.6/ilm.html)
4. [Elastic Docs ›Filebeat Reference [7.17] ›How to guides](https://www.elastic.co/guide/en/beats/filebeat/7.17/change-index-name.html)
5. [Elastic Docs ›Filebeat Reference [8.7] ›How to guides](https://www.elastic.co/guide/en/beats/filebeat/8.7/change-index-name.html)
6. [yowko/filebeat-custom-index](https://github.com/yowko/filebeat-custom-index)
