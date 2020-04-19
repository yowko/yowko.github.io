---
title: "快速建立 EFK (Elasticsearch Fluentd Kibana) 環境"
date: 2020-04-19T21:30:00+08:00
lastmod: 2020-04-19T21:30:31+08:00
draft: false
tags: ["EFK","Tools"]
slug: "docker-efk"
---

## 快速建立 EFK (Elasticsearch Fluentd Kibana) 環境

在之前筆記 [Fluentd 使用自定 Log 時間當做 Timestamp](https://blog.yowko.com/fluentd-log-time/) 中提到為了要測試 fluentd forwarder 的自訂 parser rule，所以偷懶使用 github 上的 docker-compose ([Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk))來快速建立整個環境，這幾天有個 issue 需要 debug log parser rule 的問題，發現透過這樣的方式還是相當不便利，所以我來改個版

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.5(43884)
3. docker images

    - elasticsearch:7.6.2
    - kibana:7.6.2
    - yowko/fluentd-elasticsearch:1.0.0

        > 詳細建立步驟請參考之前筆記 [Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image](https://blog.yowko.com/fluentd-elasticsearch-docker/)

## 使用方式

>大致結構還是參考 [Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk)，但預設移除 x-pack

1. 各 service 設定檔

    - ./config/elasticsearch.yml

        ```yml
        ---
        cluster.name: "docker-cluster"
        network.host: 0.0.0.0
        ```

    - ./config/fluent.conf

        ```config
        # Input
        <source>
            @type http
            port 8888
        </source>
        <source>
            @type forward
            port 24224
        </source>
        # Output
        <match *.**>
            @type elasticsearch
            host 192.168.50.97
            port 9200
            logstash_format true
        </match>
        ```

    - ./config/kibana.yml

        ```yml
        ---
        server.name: kibana
        server.host: "0"
        elasticsearch.hosts: [ "http://elasticsearch:9200" ]
        ```

2. 準備 ./dock-compose.yml

    ```yml
    version: '3.2'

    services:
      elasticsearch:
        image: elasticsearch:7.6.2
        volumes:
          - type: bind
            source: ./config/elasticsearch.yml
            target: /usr/share/elasticsearch/config/elasticsearch.yml
            read_only: true
        ports:
          - "9200:9200"
          - "9300:9300"
        environment:
          ES_JAVA_OPTS: "-Xmx256m -Xms256m"
          ELASTIC_PASSWORD: changeme
          discovery.type: single-node
        networks:
          - elk

      fluentd:
        image: yowko/fluentd-elasticsearch:1.0.0
        volumes:
          - type: bind
            source: ./config/fluent.conf
            target: /fluentd/etc/fluent.conf
        ports:
          - "24224:24224/tcp"
          - "24224:24224/udp"
          - "8888:8888"
        networks:
          - elk
        depends_on:
          - elasticsearch

      kibana:
        image: kibana:7.6.2
        volumes:
          - type: bind
            source: ./config/kibana.yml
            target: /usr/share/kibana/config/kibana.yml
            read_only: true
        ports:
          - "5601:5601"
        networks:
          - elk
        depends_on:
          - elasticsearch

    networks:
      elk:
        driver: bridge
    ```

3. 使用 docker-compose 啟動 efk

    > `--remove-orphans` 用來刪除不再存在 compose 中的資源，可加可不加

    ```bash
    docker-compose -f docker-compose.yml up -d --remove-orphans
    ```

## 心得

剛接到任務要除錯時，想說我有筆記應該可以很快把環境建起來，沒想到照著筆記還是讓我卡了好一下，這才發現之前原來我是 clone 整個官方的 git repository 回來用，我現在會覺得這樣的 dependency 太重而放棄這麼做(clone 整個 repo 回來)，這才發現原來過去我沒有這種感覺XD - 我的立場：明明只是要起個環境，又不是想了解完整內容，還大費周章 clone 整個 repo，實在很沒必要，我只想專注在 debug 我遇到的問題，環境的設定步驟愈少愈好，也就順手紀錄一下，下次應該就可以更快速地建出環境了吧XD

## 參考資訊

1. [Fluentd 使用自定 Log 時間當做 Timestamp](https://blog.yowko.com/fluentd-log-time/)
2. [Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk)
3. [Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image](https://blog.yowko.com/fluentd-elasticsearch-docker/)
