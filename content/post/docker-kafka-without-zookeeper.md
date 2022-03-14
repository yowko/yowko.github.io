---
title: "使用 Docker 啟動不依賴 ZooKeeper 的 Kafka"
date: 2022-02-07T00:30:00+08:00
lastmod: 2022-03-14T00:30:31+08:00
draft: false
tags: ["Kafka","Docker"]
slug: "docker-kafka-without-zookeeper"
---

## 使用 Docker 啟動不依賴 ZooKeeper 的 Kafka

之前筆記 [試試不依賴 ZooKeeper 的 Kafka](./kafka-without-zookeeper) 紀錄到如何使用 Kafka 的 Preview Release (KRaft mode) 功能：不再依賴 ZooKeeper，雖然已經是大幅降低了安裝與設定的難度，但還是比不上 container 的一鍵啟動，所以今天就來紀錄一下如何使用 docker compose 來快速啟動沒有 ZooKeeper 的 Kafka

## 基本環境說明

1. macOS Monterey 12.1
2. docker desktop 4.2.0(70708)
3. container images

    - quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64

## 設定方式

- server.properties

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
    ```

- docker-compose.yaml

    > 這邊需要特別留意的是使用 `root` user

    ```yaml
    version: '3'
    services:
      kafka:
        container_name: kafka
        user: root
        image: quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64
        volumes:
          - "./server.properties:/opt/kafka/config/kraft/server.properties"
        command:
          [
            "sh",
            "-c",
            "bin/kafka-storage.sh format -t $$(bin/kafka-storage.sh random-uuid) -c /opt/kafka/config/kraft/server.properties && bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties",
          ]
        ports:
          - "9092:9092"
    ```

- 未使用 `root` 的錯誤

    - `log.dirs`
        - 錯誤訊息

            ```txt
            Creating network "kafka-without-zookeeper_default" with the default driver
            Creating kafka ... done
            Attaching to kafka
            kafka    | Unable to create storage directory /opt/kafka/kafka_data: /opt/kafka/kafka_data
            kafka exited with code 1
            ```

        - 錯誤截圖

            ![1error](https://user-images.githubusercontent.com/3851540/152758025-66e1008e-7428-43ef-a438-ca03f12b1107.png)

        - 可能解決方式

            > 在 host 上建立 folder 透過 volume bind 的方式提供使用

            ```yaml
            version: '3'
            services:
              kafka:
                container_name: kafka
                image: quay.io/strimzi/kafka:latest-kafka-3.1.0-amd64
                volumes:
                  - "./server.properties:/opt/kafka/config/kraft/server.properties"
                  - type: bind
                    source: ./kafka_data
                    target: /opt/kafka/kafka_data
                command:
                  [
                    "sh",
                    "-c",
                    "bin/kafka-storage.sh format -t $$(bin/kafka-storage.sh random-uuid) -c /opt/kafka/config/kraft/server.properties && bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties",
                  ]
                ports:
                  - "9092:9092"
            ```

    - `kafkaServer-gc.log`

        > 因為不知道 `kafkaServer-gc.log` 儲存 path 的設定位置，所以我放棄 bind volume 的做法

        - 錯誤訊息

            ```txt
            Creating network "kafka-without-zookeeper_default" with the default driver
            Creating kafka ... done
            Attaching to kafka
            kafka    | Formatting kafka_data
            kafka    | mkdir: cannot create directory '/opt/kafka/bin/../logs': Permission denied
            kafka    | [0.001s][error][logging] Error opening log file '/opt/kafka/bin/../logs/kafkaServer-gc.log': No such file or directory
            kafka    | [0.001s][error][logging] Initialization of output 'file=/opt/kafka/bin/../logs/kafkaServer-gc.log' using options 'filecount=10,filesize=100M' failed.
            kafka    | Invalid -Xlog option '-Xlog:gc*:file=/opt/kafka/bin/../logs/kafkaServer-gc.log:time,tags:filecount=10,filesize=100M', see error log for details.
            kafka    | Error: Could not create the Java Virtual Machine.
            kafka    | Error: A fatal exception has occurred. Program will exit.
            kafka exited with code 1
            ```

        - 錯誤截圖

            ![2error](https://user-images.githubusercontent.com/3851540/152758038-fe592060-a934-4e81-a5ec-7ed4c906b7dd.png)

## 心得

用 root user 啟動不是很漂亮，畢竟有資安的風險，所以主流是 rootless，不過我不想多花時間去解決 folder 權限問題，畢竟用 container 啟動 kafka 的情境應該是測試或 local 開發為主，除此之外將 container 資料 mount 到 host 上會需要在每次重新啟動前都執行清除前次資料的動作，我個人覺得不是很直覺，無法享受 container 一鍵啟動(repeatable) 的好處

完整程式碼請參考 [yowko/kafka-without-zookeeper](https://github.com/yowko/kafka-without-zookeeper)

## 參考資訊

1. [試試不依賴 ZooKeeper 的 Kafka](./kafka-without-zookeeper)
2. [Three easy ways to run Kafka without Zookeeper](https://hellokube.dev/posts/three-ways-zookeepeerless-kafka/)
3. [yowko/kafka-without-zookeeper](https://github.com/yowko/kafka-without-zookeeper)
