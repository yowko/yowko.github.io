---
title: "為進到 RabbitMQ 的 message 加上時間"
date: 2021-11-08T01:30:00+08:00
lastmod: 2021-11-08T01:30:31+08:00
draft: false
tags: ["RabbitMQ"]
slug: "rabbitmq-message-timestamp"
---

## 為進到 RabbitMQ 的 message 加上時間

同事懷疑某個外部的 RabbitMQ 訊息有 delay 的狀況，想要釐清到底是外部 RabbitMQ 發送慢了還是 application 處理慢了，所以打算在 RabbitMQ 上為每個 message 加上進 queue 的時間，供後續查問題使用

原本也可以在源頭為 message 加上發送時間，但因為中間過了好幾手的 shovel，造成源頭的發送時間只能證明中間傳輸過程慢了，還是不知道慢在哪裡

## 基本環境設定

1. macOS Big Sur 11.6
2. docker desktop 3.6.0 (67351)
3. docker images

    - rabbitmq:3.8.16-management

        ```bash
        docker run -d --rm --hostname my-rabbit --name test-rabbit -p 5672:5672 -p 15672:15672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=pass.123 rabbitmq:3.8-management
        ```

4. rabbitmq plugins

    - rabbitmq_message_timestamp 3.8.0

## 設定方式

1. 啟用 `rabbitmq_message_timestamp` plugins

    > - 不需重啟 RabbitMQ
    > - 啟用時如果遇到找不到 plugin 的錯誤，請參考之前筆記 [RabbitMQ 無法啟用想要的 plugin](/rabbitmq-plugins-not-found)

    ```bash
    rabbitmq-plugins enable rabbitmq_message_timestamp
    ```

2. 實際效果

    - properties 多了 `timestamp`
    - headers 多了 `timestamp_in_ms`

    ![1timestamp](https://user-images.githubusercontent.com/3851540/140718586-563e37d1-3d18-4f57-8654-8f1e5d426677.png)

3. 說明

    - 時間資訊會儲存在 properties:`timestamp` 與 headers:`timestamp_in_ms` 中
    - 不會覆蓋原本 message 就存在的 properties:`timestamp` 或 headers:`timestamp_in_ms` 資訊

## 心得

使用上需要留意幾個情境

1. 時間資訊不是 message 第一次進 queue 的時間

    > 從進到安裝 `rabbitmq_message_timestamp` 的 instance 才會開始加上時間資訊，可能不是最源頭或是第二層

2. 只能取得第一次加入時間資訊的內容

    > 假設最源頭加上時間資訊，那麼第二層處理時並不會覆蓋，只能將原有資訊 forward，所以從第三層取得的時間資訊就是最源頭 message 進 queue 的時間

3. 如果原有 message 就會使用 properties:`timestamp` 或 headers:`timestamp_in_ms` 需要注意部份覆蓋問題

    > 只要名稱符合就會忽略不處理，但兩者是個別看的 (以下模擬原有 message 存在 properties:`timestamp`，僅寫入 headers:`timestamp_in_ms`)

    ![2overwriteone](https://user-images.githubusercontent.com/3851540/140718601-f4f5bb98-5984-4470-a662-35ae32d51214.png)

## 參考資訊

1. [Rabbitmq message arrival time stamp](https://stackoverflow.com/questions/9216712/rabbitmq-message-arrival-time-stamp)
2. [RabbitMQ 3.1.3 and the missing timestamp header](https://stackoverflow.com/questions/18002472/rabbitmq-3-1-3-and-the-missing-timestamp-header/33640262#33640262)
3. [RabbitMQ 無法啟用想要的 plugin](/rabbitmq-plugins-not-found)
