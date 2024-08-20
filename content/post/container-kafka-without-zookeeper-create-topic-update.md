---
title: "Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image 更新"
date: 2022-02-18T00:30:00+08:00
lastmod: 2024-08-19T00:30:31+08:00
draft: false
tags: ["Kafka","Container"]
slug: "container-kafka-without-zookeeper-create-topic-update"
---

## Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image 更新

<span style="color:red">建議改用官方 container image，請參考 [使用官方 image 來建立 Kafka](/docker-compose-kafka/) </style>

之前筆記 [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic-update) 紀錄到如何使用 docker compose 啟動 kraft mode (不依賴 zookeeper) 的 kafka，並在 kafka 啟動時依據 environment varaible 自動建立 topic

後來想起不同做法，立馬紀錄一下備忘

## 基本環境說明

1. macOS Monterey 12.1
2. docker desktop 4.2.0(70708)
3. container images

    - quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64

## 建立方式

1. 準備所需檔案

    - create-topic.sh

        > 依 environment varaible 來建立 topic (這個檔案與 [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic-update) 相同)

        ```bash
        #!/bin/bash

        if [[ -z "$KAFKA_CREATE_TOPICS" ]]; then
            exit 0
        fi

        if [[ -z "$START_TIMEOUT" ]]; then
            START_TIMEOUT=600
        fi

        start_timeout_exceeded=false
        count=0
        step=10
        while netstat -lnt | awk '$4 ~ /:9092$/ {exit 1}'; do
            echo "waiting for kafka to be ready"
            sleep $step;
            count=$((count + step))
            if [ $count -gt $START_TIMEOUT ]; then
                start_timeout_exceeded=true
                break
            fi
        done
        
        if $start_timeout_exceeded; then
            echo "Not able to auto-create topic (waited for $START_TIMEOUT sec)"
            exit 1
        fi
        
        IFS="${KAFKA_CREATE_TOPICS_SEPARATOR-,}"; for topicToCreate in $KAFKA_CREATE_TOPICS; do
            echo "creating topics: $topicToCreate"
            IFS=':' read -r -a topicConfig <<< "$topicToCreate"
            config=
            if [ -n "${topicConfig[3]}" ]; then
                config="--config=cleanup.policy=${topicConfig[3]}"
            fi

            COMMAND="JMX_PORT='' bin/kafka-topics.sh \\
                --create \\
                --bootstrap-server localhost:9092 \\
                --topic ${topicConfig[0]} \\
                --partitions ${topicConfig[1]} \\
                --replication-factor ${topicConfig[2]} \\
                --if-not-exists \\
                &"
            eval "${COMMAND}"

        done

        wait
        ```

    - docker-compose.yaml

        > - 需要以 `root` 為執行身份
        > - 將想要建立的 topic 透過 `{topic name}:{partition 數量}:{replication 數量}` 格式，並在需要多個 topic 時以 `,` 分隔指定至 environment variable `KAFKA_CREATE_TOPICS`
        > - 這個檔案與 [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic-update) 大致相同，只改了 image tag

        ```yaml
        version: '3'
        services:
          kafka:
            build:
              context: ./
              dockerfile: dockerfile
            container_name: kafka
            user: root
            image: yowko/kafkanozk:v2
            environment:
              - KAFKA_CREATE_TOPICS=yowkotest:3:1,yowkodemo:3:1
            ports:
              - "9092:9092"
        ```

    - dockerfile

        > 這是唯一有異動的檔案：將 `create-topic.sh` 賦予執行權限，並移除額外的啟動 shell

        ```dockerfile
        FROM quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64
        COPY ./server.properties /opt/kafka/config/kraft/server.properties
        COPY ./create-topic.sh /opt/kafka/bin/create-topic.sh
        USER root
        RUN ["chmod", "+x", "/opt/kafka/bin/create-topic.sh"]
        USER kafka
        CMD bash -c  "bin/create-topic.sh & bin/kafka-storage.sh format -t $(bin/kafka-storage.sh random-uuid) -c /opt/kafka/config/kraft/server.properties && bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties"
        ```

    - server.properties

        > 主要是 kraft mode 的設定，另外關閉自動 create topic (這個檔案與 [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic-update) 相同)

        ```config
        process.roles=broker,controller
        node.id=1
        offsets.topic.replication.factor=1
        controller.quorum.voters=1@kafka:9093
        listeners=PLAINTEXT://:9092,CONTROLLER://:9093
        advertised.listeners=PLAINTEXT://kafka:9092
        controller.listener.names=CONTROLLER
        listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
        log.dirs=kafka_data
        auto.create.topics.enable=false
        ```

2. build 與 run

    - build image 並啟動

        ```bash
        docker-compose up -d --build
        ```

    - 單純 build image

        ```bash
        docker-compose build
        ```

## 心得

這個做法的概念是：

1. 不依賴用來啟動的 shell，只使用 docker 本身的 entrypoint 來完成啟動 kafak 與建立 topic 的動作
2. 建立 topic 的 shell 先啟動並維持背景執行 (持續檢查 kafak 是否 ready)，再啟動 kafka

    > 需要保持 kafka 在前景執行，避免建立 topic 在前景而造成 topic 建立完讓 docker 認定該 container 狀態為 completed

完整程式碼請參考：[yowko/container-kafka-without-zookeeper-create-topic-update](https://github.com/yowko/container-kafka-without-zookeeper-create-topic-update)

如果不想自己 build image，也可以使用 [yowko/kafkanozk:v2](https://hub.docker.com/r/yowko/kafkanozk/tags)

## 參考資訊

1. [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic-update)
2. [yowko/container-kafka-without-zookeeper-create-topic-update](https://github.com/yowko/container-kafka-without-zookeeper-create-topic-update)
3. [yowko/kafkanozk:v2](https://hub.docker.com/r/yowko/kafkanozk/tags)
4. [使用官方 image 來建立 Kafka](/docker-compose-kafka/)
