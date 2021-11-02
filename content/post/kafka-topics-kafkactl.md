---
title: "需要 kafka-topics.sh 嗎？ 可以試試 kafkactl"
date: 2020-12-23T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["macOS","Kafka","Linux"]
slug: "kafka-topics-kafkactl"
---

## 需要 kafka-topics.sh 嗎？ 可以試試 kafkactl

因應團隊策略：資料面的異動只能透過對外的 service 來進行，不能伸手進 server，所以在 kafka 需要新增 topic 時就無法使用 kafka 內建的 kafka-topics.sh (詳細用法可以參考 [建立 Kafka 的 Topic](/create-kafka-topic))，雖然在執行建立 kafka topic 指令的 server 完整安裝 kafka 也是選項，只是單單為了需要 kafka-topics.sh 而完整安裝 Kafka 顯得不合理，所以才透過其他套件來建立 kafka topic，目前嘗試使用 [jbvmio/kafkactl](https://github.com/jbvmio/kafkactl)

## 基本環境說明

1. macOS Catalina 10.15.7
2. CentOS 8.2.2004, Kernel 4.18.0
3. Debian 10, Kernel 5.4.49+

## 安裝與使用

- 安裝 kafkactl

    >至 [GitHub release](https://github.com/jbvmio/kafkactl/releases) 下載對應版本

  - macOS

        1. 透過 homebrew 安裝

            ```bash
            brew tap jbvmio/tap && brew install jbvmio/tap/kafkactl
            ```

        2. 下載 binary

            ```bash
            curl -L https://github.com/jbvmio/kafkactl/releases/download/v1.0.26/kafkactl_1.0.26_Darwin_x86_64.tar.gz | tar -xzv
            ```

  - Debian

        ```bash
        wget -O kafkactl.deb https://github.com/jbvmio/kafkactl/releases/download/v1.0.26/kafkactl_1.0.26_linux_amd64.deb && dpkg -i kafkactl.deb 
        ```

  - CentOS

        ```bash
        wget -O kafkactl.rpm https://github.com/jbvmio/kafkactl/releases/download/v1.0.26/kafkactl_1.0.26_linux_amd64.rpm && rpm -i kafkactl.rpm
        ```

  - Linux 直接下載壓縮檔並解壓使用

        ```bash
        curl -L https://github.com/jbvmio/kafkactl/releases/download/v1.0.26/kafkactl_1.0.26_Linux_x86_64.tar.gz |tar -xzv 
        ```

- 使用 kafkactl 來新增 kafka topic

    1. 語法

        `kafkactl --broker {kafka connectionstring} admin create topic {topic name} --partitions {partition number} --replicas {replica number}`

    2. 範例

        `kafkactl --broker 10.0.16.2:9092 admin create topic yowko test --partitions 3 --replicas 1`

## 心得

`kafka-topics.sh` 有官方維護，可以保障基本功能正常，但為了要建立 topic 而安裝完整 kafka 實在是殺雞焉用牛刀，所以還是會希望可以使用簡單的 kafka client 來處理，以 [jbvmio/kafkactl](https://github.com/jbvmio/kafkactl) 這個工具來說，已經在團隊使用約半年，目前就 `建立 topic` 這個需求而言是沒有遇過問題

## 參考資訊

1. [建立 Kafka 的 Topic](/create-kafka-topic)
2. [jbvmio/kafkactl](https://github.com/jbvmio/kafkactl)
