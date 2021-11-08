---
title: "RabbitMQ 無法啟用想要的 plugin"
date: 2021-11-08T00:30:00+08:00
lastmod: 2021-11-08T00:30:31+08:00
draft: false
tags: ["RabbitMQ"]
slug: "rabbitmq-plugins-not-found"
---

## RabbitMQ 無法啟用想要的 plugin

最近有個需求打算從 RabbitMQ 來進行，需要在 RabbitMQ 安裝額外的 plugin，但熟悉的指令卻出現意外的回應，快速筆記一下處理方式

- 執行指令

    ```bash
    rabbitmq-plugins enable rabbitmq_message_timestamp
    ```

- 錯誤訊息

    ```log
    Error:
    {:plugins_not_found, [:rabbitmq_message_timestamp]}
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/140702108-a57bf547-baac-4a48-b395-78e1c93ee525.png)

- 確認 plugin 清單

    ```bash
    rabbitmq-plugins list
    ```

    ![2pluginlist](https://user-images.githubusercontent.com/3851540/140702114-d9b948ed-c076-45af-9da2-b35caac10d53.png)

## 基本環境說明

1. macOS Big Sur 11.6
2. docker desktop 3.6.0 (67351)
3. docker images

    - rabbitmq:3.8.16-management

        ```bash
        docker run -d --rm --hostname my-rabbit --name test-rabbit -p 5672:5672 -p 15672:15672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=pass.123 rabbitmq:3.8-management
        ```

4. rabbitmq plugins

    - rabbitmq_message_timestamp 3.8.0

## 安裝方式

1. 下載需要的 plugin 至 plugins 資料夾中

    ```bash
    wget -P plugins https://github.com/rabbitmq/rabbitmq-message-timestamp/releases/download/v3.8.0/rabbitmq_message_timestamp-3.8.0.ez
    ```

2. 重新列出 plugin 清單

    ```bash
    rabbitmq-plugins list
    ```

    ![3pluginlist2](https://user-images.githubusercontent.com/3851540/140702117-55f32a2a-7e85-49b4-a940-907f09252491.png)

3. 再次啟用 plugin

    ```bash
    rabbitmq-plugins enable rabbitmq_message_timestamp
    ```

    ![4enabled](https://user-images.githubusercontent.com/3851540/140702119-0e06935a-d4c5-45a8-a282-c23f6ffc0cd1.png)

## 心得

我不知道為什麼會出現找不到 plugin 的問題，以我想要安裝的 plugin `rabbitmq_message_timestamp` 為例，是 rabbitmq 官方提供的 plugin (不是來源問題)，也是從 rabbitmq 3.6.0 之後就開始 support (不是版本太舊問題)

原本以為是 docker 版本 rabbitmq 有所缺漏，但我查了實際安裝的 server 也有相同情況，只好先留個筆記來解決問題囉

## 參考資訊

1. [rabbitmq/rabbitmq-message-timestamp](https://github.com/rabbitmq/rabbitmq-message-timestamp)
2. [RabbitMQ 3.1.3 and the missing timestamp header](https://stackoverflow.com/a/33640262)
3. [RabbitMQ 3.7.0 plugins_not_found](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/issues/104#issuecomment-373319933)
