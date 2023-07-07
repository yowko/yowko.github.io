---
title: "使用 Docker Compose 啟動 Avro Schema Registry"
date: 2023-07-05T00:30:00+08:00
lastmod: 2023-07-05T00:30:31+08:00
draft: false
tags: ["docker","kafka"]
slug: "docker-compose-avro-schema-registry"
---

## 使用 Docker Compose 啟動 Avro Schema Registry

過去在使用 Kafka 時一直都沒有搭配 Avro 做序列化與反序列化，一來是偷懶，二來是都直接使用字串傳輸也方便，剛好最近看到 Kafka 序列化的介紹，趁著這個先來了解看看 Avro 的優缺點

Avro 需要 producer 與 consumer 使用相同 Avro schema 來進行資料的序列化和反序列化，而 Schema Registry 就是用來管理和存儲 Avro schema 的服務

自從 Kafka 3.3 將 KRaft mode 標記為 production ready 後，我就全面淘汰 zookerper，這次為了快速建立環境，花了點時間研究如何使用 container 來建立基於Kafka KRaft mode 的 Schema Registry，快速紀錄一下使用方式以及過程中遇到的問題

## 基本環境說明

1. macOS Ventura 13.4
2. OrbStack 0.13.0(1910)
3. container images
    - quay.io/strimzi/kafka:latest-kafka-3.5.0-amd64
    - confluentinc/cp-schema-registry:7.4.0
    - landoop/schema-registry-ui:0.9.4

## 設定方式

1. Kafka

    KRaft mode Kafka 的建立方式可以參考之前筆記 [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper/)，做法步驟跟自行安裝相同，而這次找資料時看到不同的做法：關鍵設定都可以使用 environment 來代勞，有興趣可自行參考 [KRaft Kafka Cluster with Docker](https://levelup.gitconnected.com/kraft-kafka-cluster-with-docker-e79a97d19f2c)

    剛開始嘗試 KRaft mode 時，我也試過 `confluentinc/cp-kafka` 這個 image，但當時 `confluentinc/cp-kafka` 並不支援 KRaft mode，所以後來才選擇 `strimzi/kafka`，除此之外我個人也比較偏好客製較少的 base image，想要魔改或是理解學習比較方便，但使用新版 `confluentinc/cp-kafka` 的好處是不用自己準備 config (server.properties)，只要設定 environment 就可以建立 KRaft mode Kafka

2. Schema Registry

    Confluent 官方 GitHub 的 [範例](https://github.com/confluentinc/schema-registry-workshop/blob/master/docker-compose.yml) 使用 `SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL` 這個 environment 設定，但這個是需要指向 zookeeper 的，所以這邊改用 `SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS` 就可以指向 kafka 以使用 KRaft mode

3. Schema Registry ui

    這個不是必要的，只是方便管理與確認而已，但有個坑點： environment 的 `PROXY` 需要給 `true` 才會正常連線到 Schema Registry，不然會出現 `CONNECTIVITY ERROR`

4. 完整 server.properties 與 docker-compose.yml

    - server.properties

        ```yml
        process.roles=broker,controller
        node.id=1
        offsets.topic.replication.factor=1
        controller.quorum.voters=1@kafka:9093
        listeners=PLAINTEXT://:9092,CONTROLLER://:9093
        advertised.listeners=PLAINTEXT://kafka:9092
        controller.listener.names=CONTROLLER
        listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
        log.dirs=kafka_data
        ```

    - docker-compose.yml

        ```yaml
        version: "3"
    
        services:
          kafka:
            container_name: kafka
            user: root
            image: quay.io/strimzi/kafka:latest-kafka-3.5.0-amd64
            volumes:
              - "./server.properties:/opt/kafka/config/kraft/server.properties"
            command:
              [
                "sh",
                "-c",
                "bin/kafka-storage.sh format -t $$(bin/kafka-storage.sh random-uuid) -c /opt/kafka/config/kraft/server.properties && bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties"
              ]
            ports:
              - "9092:9092"
          schema-registry:
            image: confluentinc/cp-schema-registry:7.4.0
            hostname: schema-registry
            container_name: schema-registry
            depends_on:
              - kafka
            ports:
              - "8081:8081"
            environment:
              SCHEMA_REGISTRY_HOST_NAME: schema-registry
              SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'kafka:9092'
          schema-registry-ui:
            image: landoop/schema-registry-ui
            container_name: schema-registry-ui
            depends_on:
              - schema-registry
              - kafka
            ports:
              - "8000:8000"
            environment:
              - SCHEMAREGISTRY_URL=http://schema-registry:8081
              - PROXY=true
        ```

## 心得

1. 文件不是很清楚

    [Docker hub](https://hub.docker.com/r/confluentinc/cp-schema-registry) 的第一個連結指向 [cp-docker-images](https://github.com/confluentinc/cp-docker-images/) 是一個 archived repository，follow README 的連結至 [schema-registry-images](https://github.com/confluentinc/schema-registry-images) 兩者都沒有說明可用的 environment 清單與說明

2. 也許可以改用純 docker-compose，不要再透過 file mount 來建立 Kafka

    可以參考 [KRaft Kafka Cluster with Docker](https://levelup.gitconnected.com/kraft-kafka-cluster-with-docker-e79a97d19f2c) 的做法，僅使用 docker compose 而不用自己決定 server.properties 內容

完整程式碼： [yowko/docker-compose-schema-registry](https://github.com/yowko/docker-compose-schema-registry)

## 參考資訊

1. [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper/)
2. [KRaft Kafka Cluster with Docker](https://levelup.gitconnected.com/kraft-kafka-cluster-with-docker-e79a97d19f2c)
3. Confluent 官方 GitHub 的 [範例](https://github.com/confluentinc/schema-registry-workshop/blob/master/docker-compose.yml)
4. 完整程式碼： [yowko/docker-compose-schema-registry](https://github.com/yowko/docker-compose-schema-registry)
5. [Docker hub](https://hub.docker.com/r/confluentinc/cp-schema-registry)
6. [cp-docker-images](https://github.com/confluentinc/cp-docker-images/)
7. [schema-registry-images](https://github.com/confluentinc/schema-registry-images)
