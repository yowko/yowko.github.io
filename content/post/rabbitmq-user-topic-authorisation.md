---
title: "RabbitMQ 為不同帳號設定不同 topic 權限"
date: 2021-03-14T09:30:00+08:00
lastmod: 2021-03-14T09:30:31+08:00
draft: false
tags: ["RabbitMQ","ASP.NET Core"]
slug: "rabbitmq-user-topic-authorisation"
---

## RabbitMQ 為不同帳號設定不同 topic 權限

同事在新功能的架構設計時想要讓不同 user 在 MQ 存取時可以有權限的概念，但以團隊之前使用的 Kafka 至少就我個人所知是無法達成的，不過這個需求在 RabbitMQ 上就有幾種不同的做法，今天就來紀錄一下相關的 POC 過程與設定方式

## 基本環境說明

1. macOS Big Sur 11.2.2
2. docker desktop 3.2.1 (61626)
3. docker images

    - rabbitmq:3.8.14-management

        ```bash
        docker run -d --rm --hostname my-rabbit --name test-rabbit -p 5672:5672 -p 15672:15672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=pass.123 rabbitmq:3.8.14-management
        ```

4. RABBITMQ 基本設定

    - 建立 exchange

        > 以 `yowkoex` 為例

        ```bash
        rabbitmqadmin declare exchange name=yowkoex type=topic -u admin -p pass.123
        ```

    - 建立 queue

        > 以 `topic1`，`topic2`，`topic3` 為例

        ```bash
        rabbitmqadmin declare queue name=topic1 durable=false -u admin -p pass.123
        rabbitmqadmin declare queue name=topic2 durable=false -u admin -p pass.123
        rabbitmqadmin declare queue name=topic3 durable=false -u admin -p pass.123
        ```

    - 將 queue 跟 exchange 做 binding

        ```bash
        rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="topic1" routing_key="topic1" -u admin -p pass.123
        rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="topic2" routing_key="topic2" -u admin -p pass.123
        rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="topic3" routing_key="topic3" -u admin -p pass.123
        ```

    - 建立 user

        > `yowko1` 只能 access topic1；`yowko23` 只能 access topic2 與 topic3

        ```bash
        rabbitmqctl add_user yowko1 pass.123
        rabbitmqctl add_user yowko23 pass.123
        ```

## 設定方式

> 以下三種方式擇一即可

- 使用 ui

    1. 設定 vhost 權限
    2. 設定 topic 權限

    ![1ui1](https://user-images.githubusercontent.com/3851540/111072242-a80a0080-8514-11eb-8c3e-25a51a477550.png)

    ![2ui2](https://user-images.githubusercontent.com/3851540/111072244-a9d3c400-8514-11eb-860b-089b12184b92.png)

- 使用 rabbitmqctl

    1. 設定 vhost 權限

        > 語法：`rabbitmqctl set_permissions [-p {vhost}] {user} {conf} {write} {read}s`

        ```bash
        rabbitmqctl set_permissions -p / yowko1 "" "" "topic1"
        rabbitmqctl set_permissions -p / yowko23 "" "" "^topic(2|3)$"
        ```

    2. 設定 topic 權限

        > 語法：`rabbitmqctl set_topic_permissions [-p {vhost}] {user} {exchange} {write} {read}`

        ```bash
        rabbitmqctl set_topic_permissions -p / yowko1 yowkoex "" "topic1"
        rabbitmqctl set_topic_permissions -p / yowko23 yowkoex "" "^topic(2|3)$"
        ```

- 使用 http rest api

    > 這個部份我沒有實際用過，就不特別紀錄了

## 實際效果

1. 沒有 topic 權限的錯誤

    - 錯誤訊息

        ```txt
        Unhandled exception. RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=403, text='ACCESS_REFUSED - access to queue 'topic1' in vhost '/' refused for user 'yowko23'', classId=60, methodId=20
            at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
            at RabbitMQ.Client.Impl.ModelBase.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
            at RabbitMQ.Client.Impl.AutorecoveringModel.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
            at RabbitMQ.Client.IModelExensions.BasicConsume(IModel model, String queue, Boolean autoAck, IBasicConsumer consumer)
        ```

    - 錯誤截圖

        ![3topicerror](https://user-images.githubusercontent.com/3851540/111072245-aa6c5a80-8514-11eb-896c-8c578ca5b3bd.png)

2. 沒有 vhost 權限的錯誤

    - 錯誤訊息

        ```bash
        Unhandled exception. RabbitMQ.Client.Exceptions.BrokerUnreachableException: None of the specified endpoints were reachable
         ---> RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=530, text='NOT_ALLOWED - access to vhost '/' refused for user 'yuser'', classId=10, methodId=40
           at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
           at RabbitMQ.Client.Impl.ModelBase.ConnectionOpen(String virtualHost, String capabilities, Boolean insist)
           at RabbitMQ.Client.Framing.Impl.Connection.Open(Boolean insist)
           at RabbitMQ.Client.Framing.Impl.Connection..ctor(IConnectionFactory factory, Boolean insist, IFrameHandler frameHandler, String clientProvidedName)
           at RabbitMQ.Client.Framing.Impl.AutorecoveringConnection.Init(IFrameHandler fh)
           at RabbitMQ.Client.Framing.Impl.AutorecoveringConnection.Init(IEndpointResolver endpoints)
           at RabbitMQ.Client.ConnectionFactory.CreateConnection(IEndpointResolver endpointResolver, String clientProvidedName)
           --- End of inner exception stack trace ---
           at RabbitMQ.Client.ConnectionFactory.CreateConnection(IEndpointResolver endpointResolver, String clientProvidedName)
           at RabbitMQ.Client.ConnectionFactory.CreateConnection(String clientProvidedName)
           at RabbitMQ.Client.ConnectionFactory.CreateConnection()
        ```

    - 錯誤截圖

        ![4vhosterror](https://user-images.githubusercontent.com/3851540/111072248-ab9d8780-8514-11eb-987a-4e79f05cf36d.png)

## 心得

透過 user 的權限設定可以將不同 topic 的讀寫權限分離，但這樣的做法是基於同一個 vhost 的前提下，如果想要更完整地做隔離或是需要 multiple tenants 概念就不是那麼適合，相關的做法待之後筆記再補充了

## 參考資訊

1. [rabbitmq - Docker Official Images](https://hub.docker.com/_/rabbitmq)
2. [rabbitmqctl(8)](https://www.rabbitmq.com/rabbitmqctl.8.html)
3. [Authentication, Authorisation, Access Control](https://rabbitmq.com/access-control.html)
4. [Management Command Line Tool](https://www.rabbitmq.com/management-cli.html)
5. [RabbitMQ creating queues and bindings from command line](https://stackoverflow.com/a/5826132s)
6. [RabbitMQ Management HTTP API](https://pulse.mozilla.org/api/)
