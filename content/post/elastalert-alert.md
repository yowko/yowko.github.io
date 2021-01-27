---
title: "使用 ElastAlert 監控 Elasticsearch 發出通知"
date: 2021-01-26T21:30:00+08:00
lastmod: 2021-01-26T21:30:31+08:00
draft: false
tags: ["EFK","Monitoring"]
slug: "elastalert-alert"
---

## 使用 ElastAlert 監控 Elasticsearch 發出通知

之前筆記 [使用 Docker Compose 建立 ElastAlert 測試環境](/docker-compose-elastalert) 紀錄到該如何使用 docker compose 快速建立 ElastAlert 的開發測試環境，今天要來筆記一下如何設定 ElastAlert 來發出警告

除了 Elasticsearch 外，最近也接著在進行 Prometheus 的相關設定測試，想不到過個幾天就覺得 ElastAlert 好陌生呀，想著如果不趕緊來紀錄一下  真的需要實際使用時還忘記怎麼設定就尷尬了

為了測試與驗證，我嘗試了 `email` 與 `post` 兩種方法，今天會紀錄這兩種用法的設定方式

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0 (51484)
3. Fiddler Everywhere 1.5.0
4. docker images

    - elasticsearch:7.10.1
    - yowko/fluentd-elasticsearch:1.0.0
    - kibana:7.10.1
    - praecoapp/elastalert-server:20210104

5. docker-compose

    > 詳細內容請參考之前筆記 [使用 Docker Compose 建立 ElastAlert 測試環境](/docker-compose-elastalert)

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
        #entrypoint: ["sh","/opt/elastalert-server/scripts/start.sh"]
        healthcheck:
            #test: ["CMD-SHELL", "curl -f http://localhost:3030 || exit 1"]
            test: ["CMD-SHELL", "sh /opt/elastalert-server/scripts/start.sh || exit     1"]
            interval: 30s
            timeout: 15s
            retries: 3
            start_period: 100s
    networks:
      elk:
        driver: bridge
    ```

## 設定方式

- post 設定

    1. 設定 ElastAlert rule

            ```yaml
            name: test # rule 名稱
            type: frequency # rule 的類型 (細節可以參考 [Rule Types](https://elastalert.readthedocs.io/en/latest/ruletypes.html#ruletypes))
            index: logstash-* # 監控哪個 elasticsearch index
            num_events: 1 # 命中查詢條件次數
            buffer_time: # 查詢 elasticsearch 的時間區間
            minutes: 5
            run_every: # 查詢 elasticsearch 的頻率
            minutes: 1
            timeframe: # 目標時間範圍 (60 分鐘內事件發生次數；事件內容依 type 來決定)
            minutes: 60
            realert: # 相同警告多久才發送一次避免收到過多警告，預設一分鐘，`0` 為每次警告都發送
            minutes: 0
            filter: # 查詢條件 (詳細內容可以參考 [Common Filter Types](https://elastalert.readthedocs.io/en/latest/recipes/writing_filters.html#common-filter-types))
            - query:
                query_string:
                query: "message:fluentd worker"
            alert: # 警示通知類型 (詳細內容可以參考 [Alerts](https://elastalert.readthedocs.io/en/latest/ruletypes.html#alerts))
            - "post"
            http_post_url: "http://blog.yowko.com" # post 目標 url
            http_post_proxy: "192.168.80.3:8866" # proxy 非必要設定值，這邊是我用來攔 http post 內容用的
            ```

        > 這邊我使用 Fiddler Everywhere 來攔 http post 的內容

        ![1post](https://user-images.githubusercontent.com/3851540/105938783-dfbb0700-6092-11eb-8daf-fb4396b7fbed.png)

- email 設定

    > 這邊收發 mail 透過 mailtrap service 來進行，詳細內容可以參考 [[tool]沒有mail server怎麼測試寄送email？快放過你的gmail來看看有那些可以測試用的smtp mail server](https://blog.alantsai.net/posts/2017/12/test-smtp-server-for-sending-mail-in-development)

    1. 設定 email 帳密

        > ElastAlert 使用 email 通知時，如果需要帳號密碼，需要另外用 yaml 檔案格式紀錄帳號與密碼，再將檔案位置指定給 rule 使用，以下延續之前筆記的路徑規則設定

        - elastalert/config/emailauth.yaml

            ```yaml
            user: "username"
            password: "password"
            ```

    2. 調整 docker compoe

        > 將 email 帳號密碼設定檔 mount 給 ElastAlert

        - 差異內容

                ```yaml
                - type: bind
                source: ./elastalert/config/emailauth.yaml
                target: /opt/elastalert/config/emailauth.yaml
                ```

        - elastalert 完整範例

            ```yaml
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
                #entrypoint: ["sh","/opt/elastalert-server/scripts/start.sh"]
                healthcheck:
                    #test: ["CMD-SHELL", "curl -f http://localhost:3030 || exit 1"]
                    test: ["CMD-SHELL", "sh /opt/elastalert-server/scripts/start.sh || exit 1"]
                    interval: 30s
                    timeout: 15s
                    retries: 3
                    start_period: 100s
            ```

    3. 設定 ElastAlert rule

            ```yaml
            name: test # rule 名稱 
            type: frequency # rule 的類型 (細節可以參考 [Rule Types](https://elastalert.readthedocs.io/en/latest/ruletypes.        html#ruletypes))
            index: logstash-* # 監控哪個 elasticsearch index
            num_events: 1 # 命中查詢條件次數
            buffer_time: # 查詢 elasticsearch 的時間區間
            minutes: 5
            run_every: # 查詢 elasticsearch 的頻率
            minutes: 1
            timeframe: # 目標時間範圍 (60 分鐘內事件發生次數；事件內容依 type 來決定)
            minutes: 60
            realert: # 相同警告多久才發送一次避免收到過多警告，預設一分鐘，`0` 為每次警告都發送
            minutes: 0
            filter: # 查詢條件 (詳細內容可以參考 [Common Filter Types](https://elastalert.readthedocs.io/en/latest/recipes/        writing_filters.html#common-filter-types))
            - query:
                query_string:
                query: "message:fluentd worker"
            alert: # 警示通知類型 (詳細內容可以參考 [Alerts](https://elastalert.readthedocs.io/en/latest/ruletypes.        html#alerts))
            - "email"
            email: # email 發送目標
            - "elastalert@example.com"
            alert_subject: "test alert via email" # email subject
            smtp_host: smtp.mailtrap.io
            smtp_port: 2525
            smtp_auth_file: /opt/elastalert/config/emailauth.yaml # email 帳號密碼設定檔位置
            ```

        ![2email](https://user-images.githubusercontent.com/3851540/105938792-e3e72480-6092-11eb-801c-a619476133ea.png)

## 心得

1. rule 是 `yaml` 格式，且附檔名必需為 `.yaml` 結尾，可以透過 `{elastalert ip or host}:{elastalert port}/rules` 來確認 rule 是否成功加入

    - 成功

        ![3rulesuccess](https://user-images.githubusercontent.com/3851540/105938794-e47fbb00-6092-11eb-81a7-d79ee86b5a60.png)

    - 失敗

        ![4rulefail](https://user-images.githubusercontent.com/3851540/105938795-e5185180-6092-11eb-8dba-7a3fd216a9a6.png)

2. email 帳號密碼需要額外設定檔案有點麻煩

## 參考資訊

1. [使用 Docker Compose 建立 ElastAlert 測試環境](/docker-compose-elastalert)
2. [Rule Types](https://elastalert.readthedocs.io/en/latest/ruletypes.html#ruletypes)
3. [Alerts](https://elastalert.readthedocs.io/en/latest/ruletypes.html#alerts)
4. [Common Filter Types](https://elastalert.readthedocs.io/en/latest/recipes/writing_filters.html#common-filter-types)
5. [[tool]沒有mail server怎麼測試寄送email？快放過你的gmail來看看有那些可以測試用的smtp mail server](https://blog.alantsai.net/posts/2017/12/test-smtp-server-for-sending-mail-in-development)
