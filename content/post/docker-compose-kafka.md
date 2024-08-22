---
title: "使用官方 image 來建立 Kafka"
date: 2024-08-19T00:30:00+08:00
lastmod: 2024-08-19T00:30:31+08:00
draft: false
tags: ["docker","kafka","container","docker-compose"]
slug: "docker-compose-kafka"
---

## 使用官方 image 來建立 Kafka

從 Kafka 2.8 發表 KRaft mode (KRaft early access) 以來，我就嘗試將 KRaft 導入團隊，逐步取代 Zookeeper，一開始配合官方建議： KRaft beta 功能只是在開發環境先試用，而團隊在開始環境與自動化測試環境都是使用 container 來建立的，但當時官方並未建立 container image 所以只能退而求其次使用其他第三方建立的 container image (之前筆記 [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper/)、[Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic/)、[Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image 更新](/container-kafka-without-zookeeper-create-topic-update/) 中用的都是 `strimzi/kafka`)，接著 Kafka 3.3 將 KRaft 功能標記為 production-ready、Kafka 3.5 將 ZooKeeper 列為 deprecated、Kafka 3.7 開始提供官方 container image，並且官方已經公告預計在 Kafka 4 開始不再支援 ZooKeeper

既然官方已經提供了 container image，我們就儘量使用官方提供的版本，所以今天就來紀錄一下該如何使用官方 image 來建立 Kafka，順便比較一下與之前使用第三方 image 的差異

## 基本環境說明

- macOS Sonoma 14.6.1 (Apple M2 Pro)
- OrbStack 1.6.4 (17192)
- docker images

    - apache/kafka:3.8.0

- 預設 kafka config

    > 如果沒有特別設定，會使用官方提供的預設設定：`/opt/kafka/config/server.properties` 內容如下

    <details>

    <summary style="color: LightGreen;">展開摺疊區塊</summary>
    {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "default.properties">}}
    </details>

## 使用方式

1. 全部使用預設值

    > 效果等同 `docker run -d -p 9092:9092 apache/kafka:3.8.0`

    - docker-compose.yaml

        {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "docker-compose-default.yaml">}}

    - config file

        <details>
        <summary style="color: LightGreen;">展開摺疊區塊</summary>
        {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "default.properties">}}
        </details>

2. 官方建議 [GitHub:docker/examples/docker-compose-files/single-node/plaintext/docker-compose.yml](https://github.com/apache/kafka/blob/trunk/docker/examples/docker-compose-files/single-node/plaintext/docker-compose.yml) Single Node 設定

    > 我修改了以下項目：
    > 1. 移除 version (version 已淘汰)
    > 2. image 由 {image} 改為 `apache/kafka:3.8.0`
    > 3. 移除 `CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'`

    - docker-compose.yaml

        > 我沒有找到為什麼需要 19092 port，我個人覺得沒有必要，但就忠實呈現官方建議

        {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "docker-compose-single.yaml">}}

    - config file

        <details>

        <summary style="color: LightGreen;">展開摺疊區塊</summary>
        {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "single.properties">}}
        </details>

3. 官方建議 [GitHub:docker/examples/docker-compose-files/cluster/combined/plaintext/docker-compose.yml](https://github.com/apache/kafka/blob/trunk/docker/examples/docker-compose-files/cluster/combined/plaintext/docker-compose.yml) 3 Node cluster 設定

    > 我修改了以下項目：
    > 1. 移除 version (version 已淘汰)
    > 2. image 由 {image} 改為 `apache/kafka:3.8.0`

    - docker-compose.yaml

        {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "docker-compose-cluster.yaml">}}

    - config file (以 broker 1 為例)

        <details>

        <summary style="color: LightGreen;">展開摺疊區塊</summary>
        {{<gist yowko c3780c2173033ae8a4ebed231bad4968 "cluster.properties">}}
        </details>

## 心得

相較之前使用非官方的 container image 有以下優點：

1. dockerfile 中不再需要使用 `root` 權限
2. 不用自行建立 strage directory

體驗完官方的 container image，目前沒有發現明顯的缺點，應該是可以直接轉換

## 參考資訊

1. [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper/)
2. [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image](/container-kafka-without-zookeeper-create-topic/)
3. [Build 個可以自動建立 topic 又不需依賴 ZooKeeper 的 Kafka image 更新](/container-kafka-without-zookeeper-create-topic-update/)
4. [How to run Apache Kafka without Zookeeper](https://medium.com/@erkndmrl/how-to-run-apache-kafka-without-zookeeper-3376468ddaa8)
5. [GitHub:Kafka Docker Image Usage Guide](https://github.com/apache/kafka/blob/trunk/docker/examples/README.md)
6. [GitHub:docker/examples/docker-compose-files/single-node/plaintext/docker-compose.yml](https://github.com/apache/kafka/blob/trunk/docker/examples/docker-compose-files/single-node/plaintext/docker-compose.yml)
7. [GitHub:docker/examples/docker-compose-files/cluster/combined/plaintext/docker-compose.yml](https://github.com/apache/kafka/blob/trunk/docker/examples/docker-compose-files/cluster/combined/plaintext/docker-compose.yml)
