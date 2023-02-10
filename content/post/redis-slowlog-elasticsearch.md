---
title: "使用 filebeat 將 Redis slowlog 存至 Elasticsearch"
date: 2023-02-10T00:30:00+08:00
lastmod: 2023-02-10T00:30:31+08:00
draft: false
tags: ["efk","elasticsearch","redis"]
slug: "redis-slowlog-elasticsearch"
---

## 使用 filebeat 將 Redis slowlog 存至 Elasticsearch

產品功能愈來愈多，為了存取速度的考量，對於 redis 的依存度也愈來愈高，目前團隊的 redis 是所有 application 共用一座 redis cluster，但 hash slot 的使用上不是很平均，所以某幾個 node 的壓力會大些，以致於部份存取會有 slow 的現象

雖有 slow 但又不是很常見，所以目前都是懷疑 slow 時才 request SRE 進 redis cluster 逐一將 slowlog 倒出來，看到這邊是不是覺得很 low (我也覺得)，不過有多少時間做多少事，這件事的 priority 因為發生頻率太低而持續忽略，趁著最近有空檔就來嘗試解決這個狀況

## 基本環境說明

1. macOS Ventura 13.2
2. docker desktop 4.2.0(70708)
    - Engine 20.10.10
    - Compose 1.29.2
3. docker images
    - elastic/elasticsearch:8.6.1
    - elastic/kibana:8.6.1
    - redis:7.0.8
    - docker.elastic.co/beats/filebeat:8.6.1
4. docker compose

    ```yaml
    version: "3.9"
    services:
      elasticsearch:
        image: elastic/elasticsearch:8.6.1
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
        image: kibana:8.6.1
        ports:
          - target: 5601
            published: 5601
        depends_on:
          - elasticsearch
      redis:
        image: redis:7.0.8
        ports:
          - target: 6379
            published: 6379
    volumes:
      esdata:
        driver: local
    ```

## 設定方式

1. 為 filebeat 新增 redis module 與 config

    > 檔名為 `redisbeats.yml`，下面的 docker-compose 會用到

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

2. 使用 docker compose 啟動服務

    ```yaml
    version: "3.9"
    services:
      elasticsearch:
        image: elastic/elasticsearch:8.6.1
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
        image: kibana:8.6.1
        ports:
          - target: 5601
            published: 5601
        depends_on:
          - elasticsearch
      redis:
        image: redis:7.0.8
        ports:
          - target: 6379
            published: 6379
      redisbeat:
        image: docker.elastic.co/beats/filebeat:8.6.1
        volumes:
          - ./redisbeats.yml:/usr/share/filebeat/filebeat.yml:ro
        depends_on:
          - elasticsearch
    volumes:
      esdata:
        driver: local
    ```

## 心得

- 實際效果：

    ![1result](https://user-images.githubusercontent.com/3851540/218040828-35748c84-3e94-4fe2-8140-5fd61b9025d9.png)

- 如果不熟悉 Kibana 8 與 Elasticsearch 8 的設定可以參考：[如何在 Kibana 8 與 Elasticsearch 8 上看到資料](/discover-elasticsearch8-kibana8)

設定過程中，除了 kibana 設定有卡關了，另一個就是 filebeat 的 config 設定，官網上只有部份 config 內容，所以組裝時就需要留意階層問題以及所屬的 config 群組

完整程式碼在這：[GitHub:yowko/redis-slowlog-elasticsearch](https://github.com/yowko/redis-slowlog-elasticsearch)

## 參考資訊

1. [Redis module](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-redis.html)
2. [Configure Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-reference-yml.html)
3. [如何在 Kibana 8 與 Elasticsearch 8 上看到資料](/discover-elasticsearch8-kibana8)
4. [GitHub:yowko/redis-slowlog-elasticsearch](https://github.com/yowko/redis-slowlog-elasticsearch)
