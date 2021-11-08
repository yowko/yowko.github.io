---
title: "取得 Kafka 與 zookeeper 版本"
date: 2021-11-05T00:30:00+08:00
lastmod: 2021-11-07T00:30:31+08:00
draft: false
tags: ["csharp",".NET Core","kafka"]
slug: "kafka-zookeeper-version"
---

## 取得 Kafka 與 zookeeper 版本

雖然各個環境的軟體都是使用 ansible 腳本來安裝設定的，但畢竟各個環境的執行時間有些落差，後續可能有版本更新的狀況，所以還是直接至環境上確認版本是最準確的

今天在對各個環境的服務版本時，發現我對於 kafka 跟 zookeeper 版本的查詢沒什麼印象，快速筆記一下

## 基本環境說明

1. CentOS 8 kernel 5.7.2
2. zookeeper 3.5.9
3. Kafka 2.6.2

## 取得方式

1. Kafka

    - 語法

        ```bash
        kafka-topics.sh --version
        ```

        ![1kafka](https://user-images.githubusercontent.com/3851540/140494759-42dc9ff9-2bc8-4b4d-8de8-eccb170e6462.png)

2. zookeeper

    - 語法

        ```bash
         curl -s http://{zookeeper ip}:{zookeeper admin port}/commands/envi |grep zookeeper.version
        ```

    - 範例

        > default admin port 為 `8080`，可以透過 zookeeper config (檔案位置：`{kafka_home}/config/zookeeper.properties`) 修改 `admin.serverPort={zookeeper_adminport}`

        ```bash
        curl -s http://localhost:8080/commands/envi |grep zookeeper.version
        ```

        ![2zookeeper](https://user-images.githubusercontent.com/3851540/140494764-2e872453-f207-4b9f-b12e-a99e75fbbc8b.png)

## 心得

kafka 跟 zookeeper 的版本查詢方式不太一樣，跟我之前印象不同XD  我重新檢視了一下，我推測可能跟安裝方式改變有關：過去是 zookeeper 與 kafka 獨立安裝，現在是透過整合包

## 參考資訊

1. [How to find the kafka version in linux](https://stackoverflow.com/a/51782038)
2. [Zookeeper - Where do I find the "real" version of a running instance of Zookeeper?](https://stackoverflow.com/a/69372824)
