---
title: "使用 Docker Compose 建立 ElastAlert 測試環境"
date: 2021-01-23T21:30:00+08:00
lastmod: 2021-01-23T21:30:31+08:00
draft: false
tags: ["EFK","Monitoring"]
slug: "docker-compose-elastalert"
---

## 使用 Docker Compose 建立 ElastAlert 測試環境

ElastAlert 是套基於 elasticsearch 查詢使用 python 撰寫的 open source 監控報警工具，因為在前公司用的 Elasticsearch Watcher 現在被納入 X-Pack Alerting 中，高額授權費用對於起步中的團隊是不小的負擔，所以決定先找找 open source 的工具試試或者是自行開發

今天就來紀錄一下建立 ElastAlert 測試環境的心酸血淚

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0 (51484)
3. docker images

    - elasticsearch:7.10.1
    - yowko/fluentd-elasticsearch:1.0.0
    - kibana:7.10.1
    - praecoapp/elastalert-server:20210104

## 建立方式

1. 資料夾結構

    - config
        - elasticsearch.yml
        - fluent.conf
        - kibana.yml
    - elastalert
        - config
            - elastalert.yaml
            - config.json
        - rules
        - rule_templates
    - docker-compose.yml

2. 準備 efk config
    - elasticsearch.yml

        ```yaml
        ---
        cluster.name: "docker-cluster"
        network.host: 0.0.0.0
        ```

    - fluent.conf

        ```yaml
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
            host 192.168.50.97 #elasticsearch ip，這邊使用 docker host ip
            port 9200
            logstash_format true
            user elastic #elasticsearch username
            password pass.123 #elasticsearch password
        </match>
        ```

    - kibana.yml

        ```yaml
        ---
        server.name: kibana
        server.host: "0"
        elasticsearch.hosts: ["http://192.168.50.97:9200"] #elasticsearch ip 與 port，這邊使用 docker host ip
        elasticsearch.username: "elastic" #elasticsearch username
        elasticsearch.password: "pass.123" #elasticsearch password
        ```

3. 準備 elastalert 需要的資料夾

    - config

        > 用來儲存 elastalert 與 elastalert server 設定檔

    - rules

        > 用來存放 elastalert 的 rule

    - rule_templates

        > 用來存放 elastalert rule template

4. 設定 elastalert

    - config/config.yaml

        > elastalert-server 的設定

        ```yaml
        {
          "appName": "elastalert-server",
          "port": 3030,
          "wsport": 3333,
          "elastalertPath": "/opt/elastalert",
          "verbose": true,
          "es_debug": false,
          "debug": false,
          "rulesPath": {
            "relative": true,
            "path": "/rules"
          },
          "templatesPath": {
            "relative": true,
            "path": "/rule_templates"
          },
          "es_host": "192.168.50.97", #elasticsearch ip，這邊使用 docker host ip
          "es_port": 9200,
          "es_username": "elastic", #elasticsearch username
          "es_password": "pass.123", #elasticsearch password
          "writeback_index": "elastalert_status"
        }
        ```

    - config/elastalert.yaml

        ```yaml
        es_host: 192.168.50.97 #elasticsearch ip，這邊使用 docker host ip
        es_port: 9200
        rules_folder: rules #elastalert rule 的路徑
        run_every:
          seconds: 5 # elastalert 執行間隔
        buffer_time:
          minutes: 1 # elastalert 查詢 elasticsearch 的時間
        es_username: elastic #elasticsearch username
        es_password: pass.123 #elasticsearch password
        writeback_index: elastalert_status # elastalert 的存放 index
        alert_time_limit:
          days: 2
        skip_invalid: True
        ```

5. docker-compose

    ```yaml
    version: '3.2'
    services:
      elasticsearch:
        image: elasticsearch:7.10.1
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
          discovery.type: single-node
        networks:
          - elk
        healthcheck:
            test: ["CMD-SHELL", "curl -f http://localhost:9200 || exit 1"]
            interval: 30s
            timeout: 15s
            retries: 3
            start_period: 180s
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
        image: kibana:7.10.1
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
        healthcheck:
            test: ["CMD-SHELL", "curl -f http://localhost:5601/api/status || exit 1"]
            interval: 30s
            timeout: 15s
            retries: 3
            start_period: 200s
      elastalert:
        image: praecoapp/elastalert-server
        volumes:
          - type: bind
            source: ./elastalert/config/elastalert.yaml 
            target: /opt/elastalert/config.yaml
          - type: bind
            source: ./elastalert/config/config.json
            target: /opt/elastalert-server/config/config.json
          - type: bind
            source: ./elastalert/config/emailauth.yaml
            target: /opt/elastalert/config/emailauth.yaml
          - type: bind
            source: ./elastalert/rules
            target: /opt/elastalert/rules
          - type: bind
            source: ./elastalert/rule_templates
            target: /opt/elastalert/rule_templates
        ports:
          - "3030:3030"
        networks:
          - elk
        depends_on:
          - elasticsearch
          - kibana
        entrypoint: ["sh","/opt/elastalert-server/scripts/start.sh"]
        healthcheck:
            test: ["CMD-SHELL", "curl -f http://localhost:3030 || exit 1"]
            #test: ["CMD-SHELL", "sh /opt/elastalert-server/scripts/start.sh || exit     1"]
            interval: 30s
            timeout: 15s
            retries: 3
            start_period: 100s
    networks:
      elk:
        driver: bridge
    ```

## 心得

遇到的問題

1. 無法啟動

    > 這個問題在使用 [Yelp/elastalert](https://github.com/Yelp/elastalert) 建議的 docker image (bitsensor/elastalert:latest) 出現過，後來查了 [docker run error and elasticsearch 7.7.0](https://github.com/Yelp/elastalert/issues/2995) 改用 praecoapp/elastalert-server 後解決

2. 無法建立 index

    > 這個問題，好像是從 elasticsearch 7.7 開始才遇到的，解決方式就是透過 entrypoint 或是 healthcheck 再執行一次 `start.sh`，就會重新 create index

其實問題遠不止於這兩個，其他問題因為不是在建立環境時遇到，就暫時不列在這份筆記中了，另外我自己在使用上覺得 elastalert 的穩定性並不是太好，感覺 issue 的修復速度也不太 ok，如果有其他選擇或許可以放棄 elastalert

## 參考資訊

1. [Yelp/elastalert](https://github.com/Yelp/elastalert)
2. [elastalert docs](https://elastalert.readthedocs.io/en/latest/index.html)
3. [Elasticsearchのデータを基にアラート通知する方法の調査](https://qiita.com/rururu_kenken/items/6207ab68378b70f408bb)
