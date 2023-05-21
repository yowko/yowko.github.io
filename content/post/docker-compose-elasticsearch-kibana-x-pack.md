---
title: "使用 Docker Compose 建立啟用 X-Pack 的 Elasticsearch 與 Kibana"
date: 2023-05-19T00:30:00+08:00
lastmod: 2023-05-19T00:30:31+08:00
draft: false
tags: ["efk","elasticsearch",docker]
slug: "docker-compose-elasticsearch-kibana-x-pack"
---

## 使用 Docker Compose 建立啟用 X-Pack 的 Elasticsearch 與 Kibana

EFK (Elasticsearch Fluentd/Fluent-bit Kibana) 導入團隊正式使用已經好幾年，不同需求的筆記也有幾篇，只是過去都是 focus 在 EFK 的核心功能與配套整合上，剛好最近有個專案主要是要調整 EFK 的權限設定，這才發現過去在 local 開發階段雖然是透過 docker compose 來啟動 EFK，但因為 local 沒有 security 的要求，壓根就沒有在 local docker compose 上設定 X-Pack，但這次專案是基於 X-Pack 的延伸應用，所以打算趁這個機會使用 dokcer compose 建立啟用 X-Pack 的 Elasticsearch 與 Kibana

Elasticsearch 官網上 [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) 介紹到該如何啟動 Elasticsearch container，文件末端也有說到可以使用 docker compose 啟動 Elasticsearch cluster 與 kibana，但個人認為 local 的 Elasticsearch cluster 沒有必要，不僅啟動速度較慢也較佔資源，所以才想要客製一下

## 基本環境說明

1. macOS Ventura 13.2
2. OrbStack Version 0.8.1 (1331)
3. images

    - docker.elastic.co/elasticsearch/elasticsearch:8.7.1
    - docker.elastic.co/kibana/kibana:8.7.1

## 建立方式

主要參考 [Elasticsearch 官網文件：Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

1. 建立 `.env`

    > 放 container 、 elasticsearch 與 kibana 的相關設定

    ```config
    # Password for the 'elastic' user (at least 6 characters)
    ELASTIC_PASSWORD=pass.123
    
    # Password for the 'kibana_system' user (at least 6 characters)
    KIBANA_PASSWORD=pass.123
    
    # Version of Elastic products
    STACK_VERSION=8.7.1
    
    # Set the cluster name
    CLUSTER_NAME=docker-cluster
    
    # Set to 'basic' or 'trial' to automatically start the 30-day trial
    LICENSE=basic
    #LICENSE=trial
    
    # Port to expose Elasticsearch HTTP API to the host
    ES_PORT=9200
    #ES_PORT=127.0.0.1:9200
    
    # Port to expose Kibana to the host
    KIBANA_PORT=5601
    #KIBANA_PORT=80
    
    # Increase or decrease based on the available host memory (in bytes)
    MEM_LIMIT=1073741824
    
    # Project namespace (defaults to the current folder name if not set)
    #COMPOSE_PROJECT_NAME=myproject
    ```

2. 建立 `docker-compose.yml`

    ```yaml
    version: '3'

    services:
    
      setup:
        image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
        volumes:
          - certs:/usr/share/elasticsearch/config/certs
        user: "0"
        command: >
          bash -c '
            if [ x${ELASTIC_PASSWORD} == x ]; then
              echo "Set the ELASTIC_PASSWORD environment variable in the .env file";
              exit 1;
            elif [ x${KIBANA_PASSWORD} == x ]; then
              echo "Set the KIBANA_PASSWORD environment variable in the .env file";
              exit 1;
            fi;
            if [ ! -f config/certs/ca.zip ]; then
              echo "Creating CA";
              bin/elasticsearch-certutil ca --silent --pem -out config/certs/ca.zip;
              unzip config/certs/ca.zip -d config/certs;
            fi;
            if [ ! -f config/certs/certs.zip ]; then
              echo "Creating certs";
              echo -ne \
              "instances:\n"\
              "  - name: elasticsearch\n"\
              "    dns:\n"\
              "      - elasticsearch\n"\
              "      - localhost\n"\
              "    ip:\n"\
              "      - 127.0.0.1\n"\
              > config/certs/instances.yml;
              bin/elasticsearch-certutil cert --silent --pem -out config/certs/certs.zip --in config/certs/instances.yml --ca-cert config/certs/ca/ca.crt --ca-key config/certs/ca/ca.key;
              unzip config/certs/certs.zip -d config/certs;
            fi;
            echo "Setting file permissions"
            chown -R root:root config/certs;
            find . -type d -exec chmod 750 \{\} \;;
            find . -type f -exec chmod 640 \{\} \;;
            echo "Waiting for Elasticsearch availability";
            until curl -s --cacert config/certs/ca/ca.crt https://elasticsearch:9200 | grep -q "missing authentication credentials"; do sleep 30; done;
            echo "Setting kibana_system password";
            until curl -s -X POST --cacert config/certs/ca/ca.crt -u "elastic:${ELASTIC_PASSWORD}" -H "Content-Type: application/json" https://elasticsearch:9200/_security/user/kibana_system/_password -d "{\"password\":\"${KIBANA_PASSWORD}\"}" | grep -q "^{}"; do sleep 10; done;
            echo "All done!";
          '
        healthcheck:
          test: ["CMD-SHELL", "[ -f config/certs/es01/es01.crt ]"]
          interval: 1s
          timeout: 5s
          retries: 120
    
      elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
        volumes:
          - certs:/usr/share/elasticsearch/config/certs
        ports:
          - ${ES_PORT}:9200
        environment:
          - node.name=elasticsearch
          - cluster.name=${CLUSTER_NAME}
          - cluster.initial_master_nodes=elasticsearch
          - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
          - bootstrap.memory_lock=true
          - xpack.security.enabled=true
          - xpack.security.http.ssl.enabled=true
          - xpack.security.http.ssl.key=certs/elasticsearch/elasticsearch.key
          - xpack.security.http.ssl.certificate=certs/elasticsearch/elasticsearch.crt
          - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
          - xpack.security.transport.ssl.enabled=true
          - xpack.security.transport.ssl.key=certs/elasticsearch/elasticsearch.key
          - xpack.security.transport.ssl.certificate=certs/elasticsearch/elasticsearch.crt
          - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
          - xpack.security.transport.ssl.verification_mode=certificate
          - xpack.license.self_generated.type=${LICENSE}
        mem_limit: ${MEM_LIMIT}
        ulimits:
          memlock:
            soft: -1
            hard: -1
        healthcheck:
          test:
            [
              "CMD-SHELL",
              "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
            ]
          interval: 10s
          timeout: 10s
          retries: 120
    
      kibana:
        depends_on:
          elasticsearch:
            condition: service_healthy
        image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
        volumes:
          - certs:/usr/share/kibana/config/certs
        ports:
          - ${KIBANA_PORT}:5601
        environment:
          - SERVERNAME=kibana
          - ELASTICSEARCH_HOSTS=https://elasticsearch:9200
          - ELASTICSEARCH_USERNAME=kibana_system
          - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
          - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
        mem_limit: ${MEM_LIMIT}
        healthcheck:
          test:
            [
              "CMD-SHELL",
              "curl -s -I http://localhost:5601 | grep -q 'HTTP/1.1 302 Found'",
            ]
          interval: 10s
          timeout: 10s
          retries: 120
    volumes:
      certs:
        driver: local
    ```

3. 啟動與停止

    - 啟動

        ```bash
        docker-compose -f docker-compose.yml up -d 
        ```

    - 停止

        > 記得要加上 `-v` 把建立的 volume 也清掉，避免有殘留

        ```bash
        docker-compose -f docker-compose.yml down -v
        ```

## 心得

大部份結構都與 [Elasticsearch 官網文件：Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) 相同，只是拔掉 elasticsearch cluster 的相關設定，如果不在意啟動時間與資源耗用，建立直接使用官網版本，避免我日久失修

完整程式碼請參考 [yowko/docker-compose-elasticsearch-kibana-x-pack](https://github.com/yowko/docker-compose-elasticsearch-kibana-x-pack)

## 參考資訊

1. [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
2. [yowko/docker-compose-elasticsearch-kibana-x-pack](https://github.com/yowko/docker-compose-elasticsearch-kibana-x-pack)
