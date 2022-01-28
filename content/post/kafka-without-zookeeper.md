---
title: "試試不依賴 ZooKeeper 的 Kafka"
date: 2022-01-28T00:30:00+08:00
lastmod: 2022-01-28T00:30:31+08:00
draft: false
tags: ["Kafka"]
slug: "kafka-without-zookeeper"
---

## 試試不依賴 ZooKeeper 的 Kafka

Kafka 的優異效能一直為大家所稱道，但建置部署與管理的難度也相當高，其中對於 ZooKeeper 的直接相依，不僅讓 Kafka 變得笨重也因為 ZooKeeper 的分區特性而限縮了 Kafka 整體可以負荷的水準，除此之外 ZooKeeper 本身也是套功能完善的 open source 分散式系統，其中設定的複雜度可以想見

雖然 Kafka 從 2.8 版 (Early Access Release) 開始允許不使用 ZooKeeper (KRaft mode) 直到現在的 3.1 版 (Preview Release) 都不建議使用在 production，也還有不少核心功能沒有完成，但趁著新專案啟動，我想來先嚐嚐鮮了，結果設定比想像中複雜，花點時間紀錄一下

## 基本環境說明

1. Azure VM - Standard B2s (2 vCPU,4 GiB ram)
2. Linux (centos 7.9.2009)
3. Scala 2.13
4. Kafka 3.1.0

## 安裝方式

1. 至 [kafka DOWNLOAD](https://kafka.apache.org/downloads) 下載 kafka binary
2. 解壓縮 tar

    ```bash
    mkdir kafka && tar -zxf kafka_2.13-3.1.0.tgz -C kafka --strip-components=1
    ```

3. 準備 config

    > 要啟用 kraft mode，主要是看 `process.roles`，允許的值有：
    >
    > - `broker`：傳統 kafka 的角色
    > - `controller`：做為內部 Raft quorum 的控制器
    > - `broker,controller`：同時擔任 broker 與 controller

    ```bash
    cd kafka && cat <<EOF > config/kafka.properties
    process.roles=broker,controller
    node.id=1
    controller.quorum.voters=1@$(hostname):9093
    listeners=PLAINTEXT://:9092,CONTROLLER://:9093
    advertised.listeners=PLAINTEXT://$(hostname):9092
    controller.listener.names=CONTROLLER
    listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
    log.dirs=./kafka_data
    EOF
    ```

    以下是幾個我在啟動時遇到的錯誤，提供給大家參考

    1. `node.id` 需與 `broker.id` 一致
    2. 若同時擔負 `broker,controller` 則 `listeners` 要分開設定，不然分不出來要傳什麼資訊給哪個角色
    3. `advertised.listeners` 不能包含 controller 的 listener
    4. 使用 kraft mode (`process.roles` 有設定下) `controller.quorum.voters` 需要設定有效的值

4. 建立 kafka cluster id 並建立 storage

    ```bash
     bin/kafka-storage.sh format -t $(bin/kafka-storage.sh random-uuid) -c config/kafka.properties
    ```

5. 啟動 kafka

    ```bash
    bin/kafka-server-start.sh config/kafka.properties
    ```

6. 測試與驗證

    - 建立 topic

        ```bash
        ./bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092
        ```

    - 檢查 topic

        ```bash
        bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
        ```

## 心得

原本是想快速使用 docker image 來試，但 [bitnami/bitnami-docker-kafka](https://github.com/bitnami/bitnami-docker-kafka) 與 [wurstmeister/kafka-docker](https://github.com/wurstmeister/kafka-docker) 都還不支援 kraft mode

索性先自己手動安裝，想不到手動安裝不如預期的容易，文件說明還是相當不足，但我相信在正式版推出後會慢慢變好的

## 參考資訊

1. [KRaft (aka KIP-500) mode Early Access Release](https://github.com/apache/kafka/blob/3.1/config/kraft/README.md)
2. [深度解读：Kafka 放弃 ZooKeeper，消息系统兴起二次革命](https://www.infoq.cn/article/phf3gfjutdhwmctg6kxe)
3. [How to install and configure a Kafka cluster without ZooKeeper](https://sleeplessbeastie.eu/2021/10/27/how-to-install-and-configure-a-kafka-cluster-without-zookeeper/)
