---
title: "使用 Federated Exchange 讓 RabbitMQ 跨 vhost 進行訊息傳遞"
date: 2024-12-10T00:30:00+08:00
lastmod: 2024-12-10T00:30:31+08:00
draft: false
tags: ["rabbitmq"]
slug: "rabbitmq-federated-exchange"
---

## 使用 Federated Exchange 讓 RabbitMQ 跨 vhost 進行訊息傳遞

想讓多個 consumer 可以收到同一個 RabbitMQ 訊息，你會怎麼做呢？我第一念頭是使用 Kafka 哈哈，如果限制只能用 RabbitMQ，我覺得 RabbitMQ Streams 應該能做到 (筆記可以參考：[使用 C# 存取 RabbitMQ Streams](/csharp-rabbitmq-streams/))，只是 RabbitMQ Streams 是 RabbitMQ 3.9 導入的，部份系統還在使用舊版 RabbitMQ，不一定能用，所以還有個限制是 RabbitMQ 3.8 之前版本要能用。

之前我的做法是在 message 進來的時使用 topic exchange，將 message route 至多個 queue 中：

{{<gist yowko 06b7f33d7a3374b8cd2d5861c43d3f2c "rabbitmq.sh">}}

然後再透過 shovel 將 queue 中的 message 複製到另一個 vhost 的 queue 中，細節可以參考之前筆記：[RabbitMQ Shovel 將訊息同步至不同 Vhost (Cluster)](/rabbitmq-shovel/)

雖然使用上沒有問題，但設定上就是有些繁瑣，所以今天就來紀錄一下不同用法：使用 Federated Exchange，但說實話設定上沒有比較簡單，只是在需要較多複本的情況下，Federated Exchange 看起來比較整潔，不用多個 replication 的 queue 跟多組 shovel 來同步。

做法說明：

1. 在兩個 vhost：`A` 與 `B` 各自建立一個 topic exchange：`testex`
2. 在兩個 vhost：`A` 與 `B` 各自建立一個 queue：`testq`
3. 分別將兩個 vhost：`A` 與 `B` 中的 `testq` 綁定到 `testex`，並且設定 routing key 為 `*`
4. 設定 Federated Exchange，將 `A` 的 `testex` 訊息轉傳至 `B` 的 `testex` 中

## 基本環境說明

- macOS Sequoia 15.1.1 (Apple M2 Pro)
- OrbStack Version 1.8.0 (18332)
- docker image
    - rabbitmq:3.11.9-management
    - rabbitmq:4.0.4-management
- docker-compose.yml

    {{<gist yowko 06b7f33d7a3374b8cd2d5861c43d3f2c "docker-compose.yml">}}

- RabbitMQ 前置作業

    {{<gist yowko 06b7f33d7a3374b8cd2d5861c43d3f2c "rabbitmq-setup.sh">}}

## 設定方式

- 啟用 RabbitMQ Federation 相關套件以開啟 Federated 功能

    {{<gist yowko 06b7f33d7a3374b8cd2d5861c43d3f2c "rabbitmq-plugins.sh">}}

1. 使用 web ui

    - 設定 Federated Exchange

        1. 點選 Federation Upstreams
        2. 選擇下游所在的 Virtual host `B`
        3. 設定 Federation Upstream 名稱：`A`
        4. 設定上游 URI `amqp://admin:pass.123@localhost:5672/A`
        5. 設定來源 Exchange (可以使用簡單的字中比較規則)： `testex`
        6. 設定 Queue Type 為 `quorum` (RabbitMQ 3 可能只能使用 HA policy)
        7. 加入 Upstream `Add upstream`

        - RabbitMQ 4.0.4

            ![1upstream4](https://github.com/user-attachments/assets/ea75f931-76ee-4dba-bf34-8722b962f425)
        - RabbitMQ 3.11.9

            ![2upstream3](https://github.com/user-attachments/assets/04a7e0f1-8d68-437a-8d81-6ac262596d9c)

    - 設定 Federated Policy

        1. 點選 Policies
        2. 選擇下游所在的 Virtual host `B`
        3. 設定 `Pattern` 為 exchange 名稱：`testex`
        4. 設定 Policy 名稱：`federated-exchange-policy`
        5. 選擇 Apply to `Exchanges`
        6. 設定 Proiority：`10`
        7. 設定 Definition：`{"federation-upstream-set":"all"}`
        8. 加入 Policy `Add / update policy`

        - RabbitMQ 4.0.4

            ![3policy4](https://github.com/user-attachments/assets/8b15f893-7400-46a6-8db0-fc262550a801)
        - RabbitMQ 3.11.9

            ![4policy3](https://github.com/user-attachments/assets/79f50b0b-b550-428d-bcfb-21d9476f3527)

2. 使用 rabbitmqctl

    {{<gist yowko 06b7f33d7a3374b8cd2d5861c43d3f2c "rabbitmq-federated-exchange.sh">}}

- 成功設定

    1. Federation Status 會出現 running

        ![5status](https://github.com/user-attachments/assets/22aa4d68-892b-42ac-9414-96c607a9382e)

    2. exchane 列表中上游會出現 type 為 `x-federation-upstream` 的 federated exchange，下游 exchange 會套上 policy 的 feature

        ![6exchange](https://github.com/user-attachments/assets/f3065a21-d270-4c5e-af74-ca8696055699)

    3. 上游的 exchange 詳細內容中，與 queue 的 binding 會出現 federation 的 binding

        ![8binding](https://github.com/user-attachments/assets/59630015-be7b-4007-b970-8a3b250df1bb)

    4. queue 列表中上游會出現 federation 的 queue

        ![7queue](https://github.com/user-attachments/assets/d7374fca-d392-47a1-8626-1b8930939029)

## 心得

- 版本間的 UI 有差異，造成功能的設定方式也有所不同，所以要注意版本間的差異
- 來源 vhost 如果是 root vhost，則在設定 Federated Exchange 時，不能加上 `/` 否則會出現錯誤

    ![9error](https://github.com/user-attachments/assets/8bc41a7f-fe50-483f-bbd9-23c8f3577140)

- 來源 vhost 如果是 root vhost 設定無法生效

    我試了 RabbitMQ 3.11.9 與 4.0.4，都無法設定 root vhost 的 Federated Exchange，我猜測可能是因為 root vhost 有特殊性，所以無法設定 Federated Exchange，所以如果要設定 Federated Exchange，建議使用非 root vhost

## 參考資訊

1. [Can I bind a queue from a different vhost?](https://stackoverflow.com/a/43232061)
2. [Federated Exchanges](https://www.rabbitmq.com/docs/federated-exchanges)
3. [使用 C# 存取 RabbitMQ Streams](/csharp-rabbitmq-streams/)
4. [RabbitMQ Shovel 將訊息同步至不同 Vhost (Cluster)](/rabbitmq-shovel/)
