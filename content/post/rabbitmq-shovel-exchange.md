---
title: "使用 Shovel Exchange 讓 RabbitMQ 跨 vhost 進行訊息傳遞"
date: 2024-12-24T00:30:00+08:00
lastmod: 2024-12-24T00:30:31+08:00
draft: false
tags: ["rabbitmq"]
slug: "rabbitmq-shovel-exchange"
---

## 使用 Shovel Exchange 讓 RabbitMQ 跨 vhost 進行訊息傳遞

這是之前筆記 [使用 Federated Exchange 讓 RabbitMQ 跨 vhost 進行訊息傳遞](/rabbitmq-federated-exchange/) 的延伸，雖然上次已經排除 RabbitMQ 過新的問題，但沒想到在實際使用 Federated Exchange 時，卻遇到 .NET client 在 publish message 時出現 exception 問題，我嘗試使用相同的 RabbitMQ 與 .NET client 版本，仍無法重現錯誤，又不可能無限制的讓環境出現錯誤跟其他同事支援 debug，所以我後來還是用 shovel 來解決問題

做法說明：

1. 在兩個 vhost：`A` 與 `B` 各自建立一個 topic exchange：`testex`
2. vhost `A` 建立 `q1`, `q2`,`q3` 三個 queue
3. vhost `B` 建立 `q2`,`q3` 二個 queue
4. 將 vhost `A` 的 `q1`(routing key 為 `1`), `q2`(routing key 為 `2`),`q3`(routing key 為 `3`) 三個 queue 綁定到 `testex`
5. 將 vhost `b` 的  `q2`(routing key 為 `2`),`q3`(routing key 為 `3`) 三個 queue 綁定到 `testex`
6. 設定 Shovel Exchange，將 `A` 的 `testex` 訊息轉傳至 `B` 的 `testex` 中，讓 vhost `b` 的 `q2` 與 `q3` 可以 mirror 來自 vhost `A` 的訊息，但放棄 `q1` 的訊息

## 基本環境說明

- macOS Sequoia 15.1.1 (Apple M2 Pro)
- OrbStack Version 1.8.0 (18332)
- docker image
    - rabbitmq:4.0.4-management
- docker-compose.yml

    {{<gist yowko 0311f354a9e4b2f02a60684acf146a70 "docker-compose.yml">}}

- RabbitMQ 前置作業

    {{<gist yowko 0311f354a9e4b2f02a60684acf146a70 "rabbitmq-setup.sh">}}

## 設定方式

- 啟用 RabbitMQ Federation 相關套件以開啟 Shovel 功能

    `rabbitmq-plugins enable rabbitmq_shovel rabbitmq_shovel_management`

1. 使用 web ui

    - `Shovel Management`
    - `Virtual host` 選擇 `B`
    - `Name` 設定 `fromA`
    - `Source URI` 設定 `amqp://admin:pass.123@localhost:5672/A`
    - 選擇 `Exchange`
    - `Source exchange` 選擇 `testex`
    - `Routing key` 設定 `*`
    - `Destination URI` 設定 `amqp://admin:pass.123@localhost:5672/B`
    - 選擇 `Exchange`
    - `Destination exchange` 選擇 `testex`
    - `Routing key` 保持 `` (空白)
    - `Add shovel`

2. 使用 rabbitmqctl

    > 雖然 ui 上的 destination exchange routing key 是空白，但使用 rabbitmqctl 時，`dest-exchange-key` 則不能為空白，否則無法達成想要的效果，直接不要設定即可

    ```bash
    rabbitmqctl set_parameter --vhost B shovel fromA '{"src-uri":"amqp://admin:pass.123@localhost:5672/A","src-exchange":"testex","src-exchange-key":"*","dest-uri":"amqp://admin:pass.123@localhost:5672/B","dest-exchange":"testex","add-forward-headers":true,"ack-mode":"on-confirm"}'
    ```

## 心得

跟過去使用 shovel 來 mirror queue 不同 (細節可以參考之前筆記 [RabbitMQ Shovel 將訊息同步至不同 Vhost (Cluster)](/rabbitmq-shovel/))，這次改用 shovel exchange，好處是設定方便：針對 exchange 一次搞定，缺點就是精細度不如 queue，但對於想要 mirror 所有 exchange 訊息的需求，這是個不錯的選擇。

## 參考資訊

1. [使用 Federated Exchange 讓 RabbitMQ 跨 vhost 進行訊息傳遞](/rabbitmq-federated-exchange/)
2. [RabbitMQ Shovel 將訊息同步至不同 Vhost (Cluster)](/rabbitmq-shovel/)
3. [RabbitMQ:Configuring Dynamic Shovels](https://www.rabbitmq.com/docs/shovel-dynamic)
