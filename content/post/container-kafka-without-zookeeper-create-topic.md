---
title: "Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image"
date: 2022-02-15T00:30:00+08:00
lastmod: 2024-04-19T00:30:31+08:00
draft: false
tags: ["Kafka","Container"]
slug: "container-kafka-without-zookeeper-create-topic"
---

## Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image

<span style="color:red">建議改用官方 container image，請參考 [使用官方 image 來建立 Kafka)](/docker-compose-kafka/) </style>

之前筆記 [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper) 紀錄到如何使用 docker compose 啟動 kraft mode (不依賴 zookeeper) 的 kafka

為了讓 kraft mode 的 kafka 更符合團隊的使用情境：需要在 kafka 啟動時依據 environment varaible 自動建立 topic，今天就來紀錄一下建立方式與如何使用

## 基本環境說明

1. macOS Monterey 12.1
2. docker desktop 4.2.0(70708)
3. container images

    - quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64

## 建立方式

1. 準備所需檔案

    - create-topic.sh

        > 依 environment varaible 來建立 topic (注意新建檔案時，需要有執行權限 : `chmode +x create-topic.sh`)

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

        ```yaml
        version: '3'
        services:
          kafka:
            build:
              context: ./
              dockerfile: dockerfile
            container_name: kafka
            user: root
            image: yowko/kafkanozk:v1
            environment:
              - KAFKA_CREATE_TOPICS=yowkotest:3:1,yowkodemo:3:1
            ports:
              - "9092:9092"
        ```

    - dockerfile

        ```dockerfile
        FROM quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64
        COPY ./server.properties /opt/kafka/config/kraft/server.properties
        COPY ./create-topic.sh /opt/kafka/bin/create-topic.sh
        COPY ./start.sh /opt/kafka/bin/start.sh
        ENTRYPOINT ["bin/start.sh"]
        ```

    - server.properties

        > 主要是 kraft mode 的設定，另外關閉自動 create topic

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

    - start.sh

        > 將 kafka 使用 kraft mode 啟動，並執行建立 create topic (注意新建檔案時，需要有執行權限 : `chmode +x start.sh`)

        ```bash
        #!/bin/bash

        bin/kafka-storage.sh format -t $(bin/kafka-storage.sh random-uuid) -c /opt/kafka/config/kraft/server.properties && bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties &

        bin/create-topic.sh

        # Wait for any process to exit
        wait -n
        
        # Exit with status of process that exited first
        exit $?
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

原本以為拔掉 zookeeper 後，自動建立 topic 會比較快，但實際使用上並沒有差異，以團隊的使用現況 (約莫 75 個 topic) 來看需要約 90 秒才能完成 topic 的建立，速度上差強人意但至少沒有比較慢，加上少了一顆 zookeeper container 的基本消耗跟設定，個人覺得還是可行的

完整程式碼請參考：[yowko/container-kafka-without-zookeeper-create-topic](https://github.com/yowko/container-kafka-without-zookeeper-create-topic)

如果不想自己 build image，也可以使用 [yowko/kafkanozk:v1](https://hub.docker.com/r/yowko/kafkanozk)

2022-02-18 update: 另一種寫法可以參考 [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image 更新](/container-kafka-without-zookeeper-create-topic-update)

## 參考資訊

1. [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper)
2. [yowko/container-kafka-without-zookeeper-create-topic](https://github.com/yowko/container-kafka-without-zookeeper-create-topic)
3. [docker-compose](https://docs.docker.com/compose/compose-file/compose-file-v3/)
4. [yowko/kafkanozk:v1](https://hub.docker.com/r/yowko/kafkanozk)
5. [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image 更新](/container-kafka-without-zookeeper-create-topic-update)
