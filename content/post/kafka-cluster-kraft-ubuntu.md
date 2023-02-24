---
title: "Ubuntu 安裝 Kafka KRaft cluster"
date: 2023-02-23T00:30:00+08:00
lastmod: 2023-02-23T00:30:31+08:00
draft: false
tags: ["kafka","ubuntu"]
slug: "kafka-cluster-kraft-ubuntu"
---

## Ubuntu 安裝 Kafka KRaft cluster

之前筆記 [在 CentOS 上安裝 Apache Kafka cluster](/kafka-cluster-centos) 紀錄到如何在 CentOS 上安裝基於 Zookeeper 的 Kafka cluster，最近團隊正在準備將 CentOS 以 Ubuntu 取代，另外也曾在 [試試不依賴 ZooKeeper 的 Kafka](https://blog.yowko.com/kafka-without-zookeeper/) 測試過沒有 ZooKeeper (KRaft mode) 的 kafka，當時並沒有遇到什麼問題，剛好搭上 KRaft 已經在 October 3, 2022 的 Kafka 3.3.1 被標記成 Production Ready，所以趁著這個機會，一併更新安裝語法：Ubuntu + Kraft

## 基本環境說明

1. Azure VM Standard B2s (2 vcpu，4 GiB 記憶體) X3

    - 10.1.0.4
    - 10.1.0.5
    - 10.1.0.6

2. Linux (ubuntu 22.04)
3. Kafka 3.4.0
4. OpenJDK 11 JDK

## 安裝步驟

1. 安裝 JDK

    > 版本需高於 JVM 8

    ```bash
    sudo apt install -y openjdk-11-jdk
    ```

2. 建立 user 來執行 Kafka

    > - `-U`：建立相同 group
    > - `-m`：建立 `/home/kafka`

    ```bash
    useradd --system -m -U kafka
    ```

3. 使用 kafka user 來設定

    > `su -l kafka`

    - 下載 Kafka

        > 版本資訊請參考 [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

        ```bash
        curl https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz --create-dirs -o /home/kafka/Downloads/kafka.tgz
        ```

    - 解壓縮

        ```bash
        tar -xvzf Downloads/kafka.tgz --strip 1
        ```

    - Kafka config

        > 每個 Kafka node 的 id 與 ip (advertised.listeners) 都是不同的，設定時要特別留意

        ```bash
        cat <<EOF > /home/kafka/config/kraft/server.properties
        process.roles=broker,controller
        node.id=1
        offsets.topic.replication.factor=1
        controller.quorum.voters=1@10.1.0.4:9093,2@10.1.0.5:9093,3@10.1.0.6:9093
        listeners=PLAINTEXT://:9092,CONTROLLER://:9093
        advertised.listeners=PLAINTEXT://10.1.0.4:9092
        controller.listener.names=CONTROLLER
        listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
        log.dirs=/home/kafka/kafka_data
        EOF
        ```

    - 建立 cluster id

        > 每個 node 的 cluster id 要相同

        ```bash
        kafka_cluster_id=$(./bin/kafka-storage.sh random-uuid)
        ```

    - 使用 cluster id 建立 storage

        > 如果 cluster id 不是在該 node 上建立的，記得替換成統一的 cluster id

        ```bash
        ./bin/kafka-storage.sh format -t $kafka_cluster_id -c ./config/kraft/server.properties
        ```

    - 建立 log 資料夾並退出使用 kafka user

        ```bash
        mkdir logs && exit
        ```

4. 建立 Kafka service 啟動定義

    ```bash
    cat <<EOF > /etc/systemd/system/kafka.service
    [Unit]
    Description=Kafka
    After=network.target

    [Service]
    Type=simple
    User=kafka
    ExecStart=/bin/sh -c '/home/kafka/bin/kafka-server-start.sh /home/kafka/config/kraft/server.properties > /home/kafka/logs/kafka.log 2>&1'
    ExecStop=/home/kafka/bin/kafka-server-stop.sh
    Restart=on-abnormal
    LimitNOFILE=10000000
    LimitCORE=infinity
    LimitNPROC=infinity
    LimitMEMLOCK=infinity

    [Install]
    WantedBy=multi-user.target
    EOF
    ```

5. 啟動 Kafka 服務

    ```bash
    systemctl start kafka
    ```

6. 將 Kafka 設為隨開機啟動

    ```bash
    systemctl enable kafka
    ```

## 心得

之前筆記 [試試不依賴 ZooKeeper 的 Kafka](https://blog.yowko.com/kafka-without-zookeeper/) 在 2.8 版 (Early Access Release) 中嘗試過 KRaft mode，加上也有 [在 CentOS 上安裝 Apache Kafka cluster](/kafka-cluster-centos) 的安裝經驗，原本以為會輕鬆寫意，結果東卡西卡，我沒有細究是之前的筆記沒有寫完整還是 CentOS 跟 Ubuntu 間的差異，不過終究是搞定了，相信下次需要建環境時我會很慶幸有做筆記的

## 參考資訊

1. [在 CentOS 上安裝 Apache Kafka cluster](/kafka-cluster-centos)
2. [試試不依賴 ZooKeeper 的 Kafka](https://blog.yowko.com/kafka-without-zookeeper/)
3. [APACHE KAFKA QUICKSTART](https://kafka.apache.org/quickstart)
