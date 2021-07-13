---
title: "RabbitMQ Shovel 將訊息同步至不同 Vhost (Cluster)"
date: 2021-04-05T09:30:00+08:00
lastmod: 2021-07-13T09:30:31+08:00
draft: false
tags: ["RabbitMQ"]
slug: "rabbitmq-shovel"
---

## RabbitMQ Shovel 將訊息同步至不同 Vhost (Cluster)

之前筆記 [RabbitMQ 為不同帳號設定不同 topic 權限](/rabbitmq-user-topic-authorisation) 紀錄到為了讓不同 User 在可以在收發訊息時可以控制存取權限，當時有提到透過在同個 virtual host 下設定權限的做法雖然方便，但很可能出現權限設定錯誤而造成訊息洩露或是資料遺失的狀況，今天就來紀錄一下如何透過 RabbitMQ Shovel 來設定訊息自動同步至不同 vhost 或是 cluster 的做法

## 基本環境說明

1. macOS Big Sur 11.2.2
2. docker desktop 3.2.1 (61626)
3. docker images

    - rabbitmq:3.8.14-management

        ```bash
        docker run -d --rm --hostname my-rabbit --name test-rabbit -p 5672:5672 -p 15672:15672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=pass.123 rabbitmq:3.8.14-management
        ```

4. RabbitMQ 基本設定

    > 目標如下：
    >   a. 從預設的 vhost (/) 將 sourcetopic 的訊息同步至另個新建 vhost (/yowkodown) 的 targettopic 中
    >   b. /yowko 中的 user (yowkouser) 沒有 / 的權限
    >   c. 僅同步 sourcetopic 訊息，其他訊息不會同步

    - 4.1 vhost: `/` 設定

        - 建立 exchange

            > 以 `yowkoex` 為例

            ```bash
            rabbitmqadmin declare exchange name=yowkoex type=topic -u admin -p pass.123
            ```

        - 建立 queue

            > 以 `topic1`，`topic2`，`sourcetopic` 為例

            ```bash
            rabbitmqadmin declare queue name=topic1 durable=false -u admin -p pass.123
            rabbitmqadmin declare queue name=topic2 durable=false -u admin -p pass.123
            rabbitmqadmin declare queue name=sourcetopic durable=false -u admin -p pass.123
            ```

        - 將 queue 跟 exchange 做 binding

            ```bash
            rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="topic1" routing_key="topic1" -u admin -p pass.123
            rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="topic2" routing_key="topic2" -u admin -p pass.123
            rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="sourcetopic" routing_key="sourcetopic" -u admin -p pass.123
            ```

    - 4.2 vhost: `/yowkodown` 設定

        - 建立 vhost: `/yowkodown`

            ```bash
            rabbitmqctl add_vhost yowkodown
            ```

        - 設定 `admin` 存取 `/yowkodown` 權限

            ```bash
            rabbitmqctl set_permissions -p "yowkodown" "admin" ".*" ".*" ".*"
            ```

        - 建立 exchange

            > 以 `yowkodownex` 為例

            ```bash
            rabbitmqadmin declare exchange name=yowkodownex type=topic -u admin -p pass.123 -V yowkodown
            ```

        - 建立 queue

            > 以 `targettopic` 為例

            ```bash
            rabbitmqadmin declare queue name=targettopic durable=false -u admin -p pass.123 -V yowkodown
            ```

        - 將 queue 跟 exchange 做 binding

            ```bash
            rabbitmqadmin declare binding source="yowkodownex" destination_type="queue" destination="targettopic" routing_key="targettopic" -u admin -p pass.123  -V yowkodown
            ```

        - 建立 user：`yowkouser`

            ```bash
            rabbitmqctl add_user yowkouser pass.123
            ```

        - 設定 `yowkouser` 存取權限

            > `yowkouser` 只能 access `/yowkodown`

            ```bash
            rabbitmqctl set_permissions -p "yowkodown" "yowkouser" "" "" ".*"
            ```

## 設定方式

1. 安裝 RabbitMQ Shovel plugin

    ```bash
    rabbitmq-plugins enable rabbitmq_shovel && rabbitmq-plugins enable rabbitmq_shovel_management
    ```

    ![1plugin](https://user-images.githubusercontent.com/3851540/113567411-2738b300-9641-11eb-8c6c-de008cdbc2d1.png)

    - 安裝前

        ![2noinstall](https://user-images.githubusercontent.com/3851540/113567413-2a33a380-9641-11eb-9b58-5b7a4531b9f6.png)

    - 安裝後

        ![3insttalled](https://user-images.githubusercontent.com/3851540/113567415-2a33a380-9641-11eb-8a60-7da1b9532331.png)

2. RabbitMQ Shovel 訊息同步設定 (以下四種設定方式擇一即可)

    > 從預設的 vhost (/) 將 sourcetopic 的訊息同步至另個新建 vhost (/yowkodown) 的 targettopic 中

    - 2-1. 使用 rabbitmqctl

        ```bash
        rabbitmqctl set_parameter shovel yowko_shovel '{"src-uri":"amqp://admin:pass.123@localhost:5672","src-queue":"sourcetopic","dest-uri":"amqp://admin:pass.123@localhost:5672/yowkodown","dest-queue":"targettopic","prefetch-count":64,"reconnect-delay":5,"publish-properties":[],"add-forward-headers":true,"ack-mode":"on-confirm"}'
        ```

    - 2-2. 使用 web ui

        ![4webui](https://user-images.githubusercontent.com/3851540/113567416-2acc3a00-9641-11eb-8b7d-a80f66b39627.png)

    - 2-3. 使用 restful api

        ```bash
        curl -i -u admin:pass.123 -XPUT -d'{"value":{"src-uri":"amqp://admin:pass.123@localhost:5672","src-queue":"sourcetopic","dest-uri":"amqp://admin:pass.123@localhost:5672/yowkodown","dest-queue":"targettopic","prefetch-count":64,"reconnect-delay":5,"publish-properties":[],"add-forward-headers":true,"ack-mode":"on-confirm"}}' http://localhost:15672/api/parameters/shovel/%2F/yowko_shovel
        ```

    - 2-4. 修改 `rabbitmq.config`

        > 這個需要修改 `rabbitmq.config` 暫時不符合我的使用情境，我沒有測試，需要的朋友可以參考 [Shovel Plugin](https://www.rabbitmq.com/shovel.html)

## 實際效果

![5publish](https://user-images.githubusercontent.com/3851540/113567418-2b64d080-9641-11eb-9031-325a32860de8.png)

![6consume](https://user-images.githubusercontent.com/3851540/113567420-2bfd6700-9641-11eb-9dcb-2ec283469835.png)

## 心得

可以透過安裝的套件管理工具確認狀態

![7status](https://user-images.githubusercontent.com/3851540/113567421-2bfd6700-9641-11eb-8d6b-b55ca5f5f542.png)

設定方式有三種使用上滿彈性的，設定上較難的部份個人覺得是要使用到正確的設定 key，指令的 help 中並沒有提供範例，需要額外查

今天 demo 的方式是 queue 對 queue，也可以 exchange 對 exchage 或是 queue 對 exchange，滿靈活的

## 參考資訊

1. [rabbitmq - Docker Official Images](https://hub.docker.com/_/rabbitmq)
2. [rabbitmqctl(8)](https://www.rabbitmq.com/rabbitmqctl.8.html)
3. [Authentication, Authorisation, Access Control](https://rabbitmq.com/access-control.html)
4. [Management Command Line Tool](https://www.rabbitmq.com/management-cli.html)
5. [RabbitMQ creating queues and bindings from command line](https://stackoverflow.com/a/5826132s)
6. [RabbitMQ Management HTTP API](https://pulse.mozilla.org/api/)
7. [Shovel Plugin](https://www.rabbitmq.com/shovel.html)
8. [RabbitMQ的Federation和Shovel的使用](https://zh-hant.hotbak.net/key/RabbitMQ%E7%9A%84Federation%E5%92%8C.html)
