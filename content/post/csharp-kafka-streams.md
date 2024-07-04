---
title: "使用 C# 體驗 Apache Kafka Streams"
date: 2024-07-03T00:30:00+08:00
lastmod: 2024-07-03T00:30:31+08:00
draft: false
tags: ["csharp","kafka"]
slug: "csharp-kafka-streams"
---

## 使用 C# 體驗 Apache Kafka Streams

幾年前開始使用 Kafka 時，就有注意 Kafka Streams，但當時查資料發現 Kafka Streams 是 Java 的 library (僅支援 Java 與 Scala)，並不支援 .NET，最近看技術文章時，看到有人提到使用 Kafka 來 streaming data，雖然內容跟 Apache Kafka Streams 有點不同，但讓我想起 Kafka Streams，於是再次查詢，發現有人寫了一個 .NET library - [Streamiz.Kafka.Net](https://lgouellec.github.io/kafka-streams-dotnet/)，讓 .NET 開發者也能使用 Kafka Streams，於是決定來試試看。

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack 1.6.3 (17138)
- .NET SDK 8.0.101
- JetBrains Rider 2024.1.4
- NuGet packages
    - Streamiz.Kafka.Net 1.5.1
- docker images

    - quay.io/strimzi/kafka:latest-kafka-3.7.1
    - provectuslabs/kafka-ui:v0.7.2
- 建立 kafka
    - server.properties

        > 如果要從 docker compose 外部連接到 kafka，需要在 `etc/hosts` 加上 `127.0.0.1 kafka`；如果僅在同個 docker compose 中使用，則可以省略server.properties

        {{<gist yowko 2c9919888c6019a84a9f5c66bc57c16d "server.properties">}}

    - docker-compose

        {{<gist yowko 2c9919888c6019a84a9f5c66bc57c16d "docker-compose.yml">}}

## 使用方式

- 目的：將 `from` topic 的 message 加上 `from:` 的前綴後，放入 `to` topic

- 程式碼

    {{<gist yowko 2c9919888c6019a84a9f5c66bc57c16d "Program.cs">}}

- 效果

    - from topic

        ![from](https://github.com/yowko/picsbed/assets/3851540/6d16baff-ff19-4372-a8d2-ac5efeffb32b)

    - to topic

        ![to](https://github.com/yowko/picsbed/assets/3851540/1c03f22a-51dc-4b17-8b15-8bdc14e07163)

## 心得

- 僅支援單一 kafka cluster

    ![3tips](https://github.com/yowko/picsbed/assets/3851540/f88f9cca-c28a-4a37-bc1e-911933acd8c1)

- 必填參數只有兩個：`ApplicationId` 與 `BootstrapServers`

以個人粗淺的看法，我覺得 Kafka Streams 就是將某個 topic 的內容取出來，進行處理後再放回另一個 topic，這樣的概念，事實上跟過去使用 producer 與 consumer 相比並沒有多大的差別，Kafka Streams 提供更高層次的抽象而將 producer 與 consumer 做了封裝

介紹文件：[Streamiz.Kafka.Net](https://lgouellec.github.io/kafka-streams-dotnet/) 還有不少功能，初步看來應該可以符合不同的需求，未來有機會再深入研究

## 參考資訊

1. [Apache Kafka Streams](https://kafka.apache.org/documentation/streams)
2. [GitHub:LGouellec/kafka-streams-dotnet](https://github.com/LGouellec/kafka-streams-dotnet)
3. [Streamiz.Kafka.Net](https://lgouellec.github.io/kafka-streams-dotnet/)
