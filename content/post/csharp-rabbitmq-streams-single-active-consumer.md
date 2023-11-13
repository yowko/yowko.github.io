---
title: "使用 C# 設定 Single Active Consumer 讀取 RabbitMQ Streams"
date: 2023-11-07T00:30:00+08:00
lastmod: 2023-11-10T00:30:31+08:00
draft: false
tags: ["csharp","rabbitmq"]
slug: "csharp-rabbitmq-streams-single-active-consumer"
---

## 使用 C# 設定 Single Active Consumer 讀取 RabbitMQ Streams

RabbitMQ 團隊在 [RabbitMQ 3.9](https://www.rabbitmq.com/streams.html) 導入 `Streams`，官網文件大致上說明了有哪些異動與效能改善，以下整理個人理解：

1. 不會像過去 RabbitMQ 在 message 得到 ack 後就刪除，處理方式如同 Kafka
2. 提供 server-side offset 追蹤，讓 consumer 可以從上次 offset 繼續消費，這也跟 Kafka 一樣
3. 可以使用二進位協定來存取，有效提升效能

根據官網說明，RabbitMQ Streams 適合的情境有：

1. 多個應用程式需要存取相同的資料
2. 大量的資料需要被存取 (RabbitMQ Streams 將訊息儲存在 disk 而非傳統 queue 的 memory)
3. 訊息的 replay (可以透過 offset 或是指定 timestamp 來重新讀取過去的訊息)
4. 高傳輸量的應用程式 (相較於傳統 queue 高出幾個數量級)

之前筆記 [使用 C# 存取 RabbitMQ Streams](/csharp-rabbitmq-streams) 紀錄的是 RabbitMQ Stream queue type 的使用方式，並且使用 C# 搭配 RabbitMQ .NET client  來存取，也在 [使用 C# 透過二進位協定存取 RabbitMQ Streams](/csharp-rabbitmq-streams-binary) 紀錄到如何使用 RabbitMQ Stream .Net Client 透過二進位協定的存取方式，今天要來紀錄如何讓 RabbitMQ Streams 如同 Kafka consumer group 的效果：只有一個 consumer 在消費訊息

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 1.0.1(16297)
3. .NET SDK 6.0.413
4. JetBrains Rider 2023.2.3
5. NuGet libaries

    - RabbitMQ.Stream.Client 1.7.4
    - Microsoft.Extensions.Logging.Console 7.0.0

6. docker images
   - rabbitmq:3.12.7-management
   - yowko/rabbitmq:3.12.7-management
7. RabbitMQ cluster docker compose

    > 詳細內容可以參考過去筆記 [透過 docker compose 啟動 RabbitMQ cluster](/docker-compose-rabbitmq-cluster/)

    {{<gist yowko 95998a9f62af6dc2ff1b55c40013a798>}}

8. 建立 RabbitMQ Stream queue

    > 使用 binary protocol 可以不用建立 exchange 與 binding

    ```bash
    rabbitmqadmin declare queue name=test-streams durable=true queue_type=stream -u admin -p pass.123
    ```

## 設定方式：設定 Consumer Config

1. 設定 Reference 名稱

    `Reference = "yowkoconsumer"`

2. 啟用單一 active consumer

    `IsSingleActiveConsumer = true`

3. 取得 Reference 對應 offset：加上 `ConsumerUpdateListener`

    > 這邊我個人有稍做修改：在使用 reference 與 stream 查不到相關紀錄時不拋出 `OffsetNotFoundException`，而是回傳 `OffsetTypeFirst` 讓 consumer 從第一筆開始消費

    ![2consumerlistener](https://github.com/yowko/picsbed/assets/3851540/1144a780-3f79-4d0b-b58f-eb8360d0b4be)

    ```csharp
    ConsumerUpdateListener = async (consumerRef, stream, isActive) =>
    {
        try
        {
            var offset = await streamSystem.QueryOffset(consumerRef, stream).ConfigureAwait(false);
            return new OffsetTypeOffset(offset);
        }
        catch (Exception e) when (e is OffsetNotFoundException)
        {
            Console.WriteLine(e);
            return new OffsetTypeFirst();
        }
    }
    ```

4. 儲存 offfset 追蹤：修改 `MessageHandler`

    > 嚴格來說，這個動作不屬於 single active consumer 的範疇，但是如果不設定的話，server 就沒有辦法知道 consumer 已經消費到哪裡，算是 single active consumer 的必要前提

    ```cs
    MessageHandler = async (sourceStream, consumer, messageContext, message) =>
    {
        Console.WriteLine(
            $"Received message: {Encoding.ASCII.GetString(message.Data.Contents)} |{messageContext.Offset} | {messageContext.Timestamp.TotalMilliseconds}");
        await consumer.StoreOffset(messageContext.Offset).ConfigureAwait(false); 
        await Task.CompletedTask.ConfigureAwait(false);
    }
    ```

5. 完整 consumer 內容

    {{< gist yowko 8693a6b1f2e1b009a1fac046b5cc5e9a>}}

## 心得

1. RabbitMQ Stream 的 consumer group 設定方式與 Kafka 不同，Kafka 是透過 group id 來設定，而 RabbitMQ Stream 則是透過 Reference 來設定
2. RabbitMQ Stream single active consumer 的設定明顯比 Kafka consumer group 繁瑣，需要自行設定 offset 追蹤與儲存，相較需 kafka 只要指定 group id 其他的皆交由 kafka 處理
3. 個人實測，在沒有新增 message 的情況下，雖然有回傳最後一筆 message 內容，但重啟 consumer 時還是會收到最後一筆 message (這筆 message 會重複收到)
4. 設定 ConsumerUpdateListener 時，如果 Reference 與 stream 不存在時，會拋出 `OffsetNotFoundException`，我修改官網的範例以回傳 `OffsetTypeFirst` 讓 consumer 從第一筆開始消費 (我覺得這邊可以改善，可以直接回傳 `OffsetTypeFirst`，不需要拋出例外，否則以我看就是會一直收到 `OffsetNotFoundException`，還是我誤會了什麼XD)

    - 錯誤訊息

        ```log
        15:11:00 fail: RabbitMQ.Stream.Client.Reliable.Consumer[0] Error reading the socket RabbitMQ.Stream.Client.OffsetNotFoundException: QueryOffset stream: test-streams, reference: yowkoconsumer    at RabbitMQ.Stream.Client.ClientExceptions.MaybeThrowException(ResponseCode responseCode, String message) in /_/RabbitMQ.Stream.Client/ClientExceptions.cs:line 25    at RabbitMQ.Stream.Client.StreamSystem.QueryOffset(String reference, String stream) in /_/RabbitMQ.Stream.Client/StreamSystem.cs:line 343    at Program.<>c__DisplayClass0_0.<<<Main>$>b__3>d.MoveNext() in /Users/yowko.tsai/POCs/StreamBinaryDemo/StreamBinaryDemo/Program.cs:line 59 --- End of stack trace from previous location ---    at RabbitMQ.Stream.Client.RawConsumer.<>c__DisplayClass17_0.<<Init>b__2>d.MoveNext() in /_/RabbitMQ.Stream.Client/RawConsumer.cs:line 536 --- End of stack trace from previous location ---    at RabbitMQ.Stream.Client.Client.HandleIncoming(Memory`1 frameMemory) in /_/RabbitMQ.Stream.Client/Client.cs:line 495    at RabbitMQ.Stream.Client.Connection.ProcessIncomingFrames() in /_/RabbitMQ.Stream.Client/Connection.cs:line 163
        ```

    - 錯誤截圖

        ![1error](https://github.com/yowko/picsbed/assets/3851540/b237a0bd-a091-40d3-8096-6a107a74cbcd)

完整程式碼：[yowko/rabbitmq-streams-binary](https://github.com/yowko/rabbitmq-streams-binary)

## 參考資訊

1. [RabbitMQ 3.9](https://www.rabbitmq.com/streams.html)
2. [使用 C# 存取 RabbitMQ Streams](/csharp-rabbitmq-streams)
3. [使用 C# 透過二進位協定存取 RabbitMQ Streams](/csharp-rabbitmq-streams-binary)
4. [RabbitMQ Stream .Net Client](https://rabbitmq.github.io/rabbitmq-stream-dotnet-client/stable/htmlsingle/index.html)
5. [yowko/rabbitmq-streams-binary](https://github.com/yowko/rabbitmq-streams-binary)
