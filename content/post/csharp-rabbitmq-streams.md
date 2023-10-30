---
title: "使用 C# 存取 RabbitMQ Streams"
date: 2023-10-30T00:30:00+08:00
lastmod: 2023-10-30T00:30:31+08:00
draft: false
tags: ["csharp","rabbitmq"]
slug: "csharp-rabbitmq-streams"
---

## 使用 C# 存取 RabbitMQ Streams

RabbitMQ 團隊在 [RabbitMQ 3.9](https://www.rabbitmq.com/streams.html) 導入 `Streams`，官網文件大致上說明了有哪些異動與效能改善，以下整理個人理解：

1. 不會像過去 RabbitMQ 在 message 得到 ack 後就刪除，處理方式如同 Kafka
2. 提供 server-side offset 追蹤，讓 consumer 可以從上次 offset 繼續消費，這也跟 Kafka 一樣
3. 可以使用二進位協定來存取，有效提升效能

根據官網說明，RabbitMQ Streams 適合的情境有：

1. 多個應用程式需要存取相同的資料
2. 大量的資料需要被存取 (RabbitMQ Streams 將訊息儲存在 disk 而非傳統 queue 的 memory)
3. 訊息的 replay (可以透過 offset 或是指定 timestamp 來重新讀取過去的訊息)
4. 高傳輸量的應用程式 (相較於傳統 queue 高出幾個數量級)

今天要來紀錄的是 RabbitMQ Stream queue type 的使用方式，並且使用 C# 來存取，與二進位協定的存取方式不同(需要使用 stream client，這邊使用的是 RabbitMQ .NET client 來存取，除此之外 RabbitMQ 也不需額外安裝 stream 相關 plugin)。

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 1.0.1(1697)
3. NuGet librearies

    - RabbitMQ.Client 6.6.0

4. docker images
   - rabbitmq:3.12.7-management
   - yowko/rabbitmq:3.12.7-management
5. RabbitMQ cluster docker compose

    > 詳細內容可以參考過去筆記 [透過 docker compose 啟動 RabbitMQ cluster](/docker-compose-rabbitmq-cluster/)

    {{<gist yowko 95998a9f62af6dc2ff1b55c40013a798>}}

6. 建立 RabbitMQ exchange

    ```bash
    rabbitmqadmin declare exchange name=test type=topic -u admin -p pass.123
    ```

7. 建立 RabbitMQ Stream queue

    ```bash
    rabbitmqadmin declare queue name=test-streams durable=true queue_type=stream -u admin -p pass.123
    ```

8. 設定 exchange queue 的 binding

    ```bash
    rabbitmqadmin declare binding source=test destination_type=queue destination=test-streams routing_key=streams -u admin -p pass.123
    ```

## C# 存取 RabbitMQ Stream

1. Publish

    {{< gist yowko 8fdfd3fa77e02298fafa08fe36568523 >}}

2. Consume

    {{< gist yowko 5dd23f8a32f3b7b9b12927ed7c2b7c6f >}}

## 心得

1. RabbitMQ 無需啟用 `rabbitmq_stream` 與 `rabbitmq_stream_management` plugin
2. 不需使用 [RabbitMQ.Stream.Client](https://www.nuget.org/packages/RabbitMQ.Stream.Client) 來存取
3. 必需設定 QOS (有些設定不支援)

    - QOS 未設定錯誤訊息

        ```log
        Unhandled exception. RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=406, text='PRECONDITION_FAILED - consumer prefetch count is not set for 'queue 'test-streams' in vhost '/''', classId=60, methodId=20
           at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
           at RabbitMQ.Client.Impl.ModelBase.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
           at RabbitMQ.Client.Impl.AutorecoveringModel.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
           at RabbitMQ.Client.IModelExensions.BasicConsume(IModel model, IBasicConsumer consumer, String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments)
           at Program.<<Main>$>g__ConsumeStreams|0_1(IConnection connection, IModel channel, EventingBasicConsumer consumer) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 67
           at Program.<Main>$(String[] args) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 47
        ```

        ![1noqos](https://github.com/yowko/picsbed/assets/3851540/c66b80da-135a-4c19-a25d-16ae780cf501)

    - prefetchSize `只能設為 0`

        ```log
        Unhandled exception. RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=540, text='NOT_IMPLEMENTED - prefetch_size!=0 (1)', classId=60, methodId=10
           at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
           at RabbitMQ.Client.Impl.ModelBase.ModelRpc(MethodBase method, ContentHeaderBase header, Byte[] body)
           at RabbitMQ.Client.Framing.Impl.Model.BasicQos(UInt32 prefetchSize, UInt16 prefetchCount, Boolean global)
           at RabbitMQ.Client.Impl.AutorecoveringModel.BasicQos(UInt32 prefetchSize, UInt16 prefetchCount, Boolean global)
           at Program.<Main>$(String[] args) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 29

        Process finished with exit code 134.
        ```

        ![2prefetchsizeonly0](https://github.com/yowko/picsbed/assets/3851540/1c213bdc-0038-41e3-9674-8a8e29dda3b4)

    - prefetchCount `不得設為 0`

        ```log
        Unhandled exception. RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=406, text='PRECONDITION_FAILED - consumer prefetch count is not set for 'queue 'test-streams' in vhost '/''', classId=60, methodId=20
           at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
           at RabbitMQ.Client.Impl.ModelBase.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
           at RabbitMQ.Client.Impl.AutorecoveringModel.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
           at RabbitMQ.Client.IModelExensions.BasicConsume(IModel model, IBasicConsumer consumer, String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments)
           at Program.<<Main>$>g__ConsumeStreams|0_1(IConnection connection, IModel channel, EventingBasicConsumer consumer) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 67
           at Program.<Main>$(String[] args) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 47

        Process finished with exit code 134.
        ```

        ![3prefetchcountnot0](https://github.com/yowko/picsbed/assets/3851540/538412bf-2fa9-400f-b158-a997040dad58)

    - global `只能設為 false`

        ```log
        Unhandled exception. RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=406, text='PRECONDITION_FAILED - consumer prefetch count is not set for 'queue 'test-streams' in vhost '/''', classId=60, methodId=20
           at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
           at RabbitMQ.Client.Impl.ModelBase.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
           at RabbitMQ.Client.Impl.AutorecoveringModel.BasicConsume(String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments, IBasicConsumer consumer)
           at RabbitMQ.Client.IModelExensions.BasicConsume(IModel model, IBasicConsumer consumer, String queue, Boolean autoAck, String consumerTag, Boolean noLocal, Boolean exclusive, IDictionary`2 arguments)
           at Program.<<Main>$>g__ConsumeStreams|0_1(IConnection connection, IModel channel, EventingBasicConsumer consumer) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 67
           at Program.<Main>$(String[] args) in /Users/yowko.tsai/POCs/Solution1/RabbitMqPublishConsume/Program.cs:line 47

        Process finished with exit code 134.
        ```

        ![4globalonlyfalse](https://github.com/yowko/picsbed/assets/3851540/ca0c54a6-f8ed-4ad5-8bcd-506cfaee0194)

4. x-stream-offset(stream-offset) offset 與 timestamp 使用上模糊

    文件說允許 `"first"`、`"last"`、`"next"`、`Offset`、`Timestamp`、`Interval`，但實際測試下來 `Timestamp` 沒有成功過，感覺都是吃到 `Offset` 的效果，不知道是不是我個人對文件的理解有問題

    ![5streamoffset](https://github.com/yowko/picsbed/assets/3851540/a9c7dcf1-11d1-4d6b-b7c5-9c2f85728fa4)

5. 文件不夠充足

    這個可能是我個人問題：queue type: stream 與 binary RabbitMQ Stream protocol 區別不夠明顯，範例與說明容易混淆，而且有些設定不支援，但文件沒有明確說明，只能透過測試才知道

6. 雖然是對應 Kafka 相關功能，但還是有不少差異，其他像是 Kafka consumer group 的設定，RabbitMQ 可能需要透過 single active consumer 實作、RabbitMQ 有 Super streams，初步看來跟 Kafka partition 概念接近，但都還沒花時間仔細研究

## 參考資料

1. [RabbitMQ 3.9](https://www.rabbitmq.com/streams.html)
2. [透過 docker compose 啟動 RabbitMQ cluster](/docker-compose-rabbitmq-cluster/)
3. [RabbitMQ Streams Overview](https://blog.rabbitmq.com/posts/2021/07/rabbitmq-streams-overview/)
4. [RabbitMQ.Stream.Client](https://www.nuget.org/packages/RabbitMQ.Stream.Client)
