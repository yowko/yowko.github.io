---
title: "透過 container 啟動 Pulsar Manager"
date: 2022-03-13T00:30:00+08:00
lastmod: 2022-03-13T00:30:31+08:00
draft: false
tags: ["Pulsar"]
slug: "pulsar-manager"
---

## 透過 container 啟動 Pulsar Manager

Message Queue 在正式服務上運行時大部份不會啟用 GUI，甚至像 kafka 原生就沒有提供，不過開發階段有 GUI 來協助確認訊息或是設定還是便利不少

- RabbitMQ 有官方 plugin：`management`
- Kafka 有外部公司產品：`Conduktor`
- Pulsar 也有官方 web： `Pulsar Manager`

今天就來紀錄一下使用 docker 啟動 `Pulsar Manager` 與使用方式

## 基本環境說明

1. macOS Monterey 12.2.1
2. docker desktop 4.2.0(70708)
3. docker images

    - apachepulsar/pulsar:2.9.1
    - apachepulsar/pulsar-manager:v0.2.0

4. 使用 docker 啟動 pulsar

    ```bash
    docker run -d -p 6650:6650 -p 8080:8080 --name pulsar apachepulsar/pulsar:latest bin/pulsar standalone
    ```

    - `6650` 是 broker service port
    - `8080` 是 web service port

## 設定與使用方式

1. 使用 docker 啟動

    ```bash
    docker run -d -p 9527:9527 -p 7750:7750 -e SPRING_CONFIGURATION_FILE=/pulsar-manager/pulsar-manager/application.properties apachepulsar/pulsar-manager:v0.2.0
    ```

    - `9527` 是 web ui 的 port
    - `7750` 是後台 api 的 port

2. 建立 user

    - 語法

        ```bash
        CSRF_TOKEN=$(curl http://localhost:7750/pulsar-manager/csrf-token) &&
        curl \
           -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \
           -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
           -H "Content-Type: application/json" \
           -X PUT http://localhost:7750/pulsar-manager/users/superuser \
           -d '{"name": "{username}", "password": "{password}", "email": "{email}"}'
        ```

    - 範例

        ```bash
        CSRF_TOKEN=$(curl http://localhost:7750/pulsar-manager/csrf-token) &&
        curl \
           -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \
           -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
           -H "Content-Type: application/json" \
           -X PUT http://localhost:7750/pulsar-manager/users/superuser \
           -d '{"name": "yowko", "password": "pass.123",  "email": "yowko@yowko.com"}'
        ```

3. 登入成功後，需要先建立 pulsar 的 endpoint

    - New Environment

        ![1newenvironment](https://user-images.githubusercontent.com/3851540/158127314-14793818-d9b7-4aa0-a9bc-531b279f4031.png)

    - Name and URL

        > 因為使用 docker 啟動，記得 url 不要填 localhost or 127.0.0.1 會連到 Pulsar Manager 本身

        ![2nameurl](https://user-images.githubusercontent.com/3851540/158127326-dc59d3cb-c229-4edd-b324-ff25e1b80a82.png)

    - 成功新增

        ![3success](https://user-images.githubusercontent.com/3851540/158127331-2bd9c4f1-eaf6-408f-b390-42acfb366440.png)

        - 若 url 無法連線會出現錯誤訊息

            ![4error](https://user-images.githubusercontent.com/3851540/158127335-ceb6f418-ae86-4cec-a1a6-188edfee2713.png)

        - 若未填 schema 則畫面不會有任何動作

            ![5noschema](https://user-images.githubusercontent.com/3851540/158127339-9a6746e7-d565-4237-87f6-7b140d6b7dfe.png)

## 心得

Pulsar Manager 的 UI 有些地方 ux 不是那麼直覺

1. 像是前面提到的 Environment 設定，沒有填 schema 卻沒有反應
2. 後面 New Topic 如果 pulsar 無法連線也不會提示，就是畫面沒有反應而已
3. 部份功能 tab 也許是 cache，點來點去不如預期  後來才發現是 session 過期
4. 我沒找到直接 consume 訊息的功能

## 參考資訊

1. [Pulsar Manager](https://pulsar.apache.org/docs/en/administration-pulsar-manager/)
