---
title: "使用 C# 透過二進位協定存取 RabbitMQ Streams"
date: 2023-11-03T00:30:00+08:00
lastmod: 2023-11-06T00:30:31+08:00
draft: false
tags: ["csharp","rabbitmq"]
slug: "csharp-rabbitmq-streams-binary"
---

## 使用 C# 透過二進位協定存取 RabbitMQ Streams

RabbitMQ 團隊在 [RabbitMQ 3.9](https://www.rabbitmq.com/streams.html) 導入 `Streams`，官網文件大致上說明了有哪些異動與效能改善，以下整理個人理解：

1. 不會像過去 RabbitMQ 在 message 得到 ack 後就刪除，處理方式如同 Kafka
2. 提供 server-side offset 追蹤，讓 consumer 可以從上次 offset 繼續消費，這也跟 Kafka 一樣
3. 可以使用二進位協定來存取，有效提升效能

根據官網說明，RabbitMQ Streams 適合的情境有：

1. 多個應用程式需要存取相同的資料
2. 大量的資料需要被存取 (RabbitMQ Streams 將訊息儲存在 disk 而非傳統 queue 的 memory)
3. 訊息的 replay (可以透過 offset 或是指定 timestamp 來重新讀取過去的訊息)
4. 高傳輸量的應用程式 (相較於傳統 queue 高出幾個數量級)

之前筆記 [使用 C# 存取 RabbitMQ Streams](/csharp-rabbitmq-streams) 紀錄的是 RabbitMQ Stream queue type 的使用方式，並且使用 C# 搭配 RabbitMQ .NET client  來存取，今天則是要來紀錄使用二進位協定的存取方式(需要使用 RabbitMQ Stream .Net Client)

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 1.0.1(1697)
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

## 設定方式

1. 安裝 `rabbitmq_stream` rabbitmq plugin

    > `rabbitmq_stream_management` 則可有可無，主要是用來觀察 stream 的 connection
    >
    >![2streammgt](https://github.com/yowko/picsbed/assets/3851540/1e8e647f-0f66-4ec6-82ab-85229e60e7db)

    ```bash
    rabbitmq-plugins enable rabbitmq_stream
    ```

    - 未安裝錯誤訊息

        ```log
        16:04:37 fail: RabbitMQ.Stream.Client.StreamSystem[0] Error connecting to 127.0.0.1:5552. Trying next endpoint System.ObjectDisposedException: Cannot access a disposed object. Object name: 'System.Net.Sockets.NetworkStream'.    at System.Net.Sockets.NetworkStream.<ThrowIfDisposed>g__ThrowObjectDisposedException|63_0()    at System.Net.Sockets.NetworkStream.WriteAsync(ReadOnlyMemory`1 buffer, CancellationToken cancellationToken)    at System.IO.Pipelines.StreamPipeWriter.FlushAsyncInternal(Boolean writeToStream, ReadOnlyMemory`1 data, CancellationToken cancellationToken)    at RabbitMQ.Stream.Client.Connection.WriteCommand[T](T command) in /_/RabbitMQ.Stream.Client/Connection.cs:line 123    at RabbitMQ.Stream.Client.Connection.Write[T](T command) in /_/RabbitMQ.Stream.Client/Connection.cs:line 95    at RabbitMQ.Stream.Client.Client.Request[TIn,TOut](Func`2 request, Nullable`1 timeout) in /_/RabbitMQ.Stream.Client/Client.cs:line 407    at RabbitMQ.Stream.Client.Client.Create(ClientParameters parameters, ILogger logger) in /_/RabbitMQ.Stream.Client/Client.cs:line 214    at RabbitMQ.Stream.Client.StreamSystem.Create(StreamSystemConfig config, ILogger`1 logger) in /_/RabbitMQ.Stream.Client/StreamSystem.cs:line 69
        16:04:37 fail: RabbitMQ.Stream.Client.StreamSystem[0] Error reading the socket System.IO.IOException: Unable to read data from the transport connection: Connection reset by peer.  ---> System.Net.Sockets.SocketException (54): Connection reset by peer    --- End of inner exception stack trace ---    at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.ThrowException(SocketError error, CancellationToken cancellationToken)    at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.System.Threading.Tasks.Sources.IValueTaskSource<System.Int32>.GetResult(Int16 token)    at System.IO.Pipelines.StreamPipeReader.<ReadAsync>g__Core|36_0(StreamPipeReader reader, CancellationTokenSource tokenSource, CancellationToken cancellationToken)    at RabbitMQ.Stream.Client.Connection.ProcessIncomingFrames() in /_/RabbitMQ.Stream.Client/Connection.cs:line 142

        ```

        ![1nopluginerror](https://github.com/yowko/picsbed/assets/3851540/4ad34f4c-7a57-45e1-93c1-62e87f2ea484)

2. Produce

    {{<gist yowko 9c37b156a5439979da7fb037397a012b >}}

3. Consume

    {{<gist yowko 0a003a7b02407e8914ff7ea9afe57c51 >}}

## 心得

RabbitMQ.Stream.Client 的方法與 RabbitMQ.Client 有些許不同，使用上需要熟悉一下，以下整理一些個人的心得：

1. RabbitMQ.Stream.Client 方法大多是非同步的
2. RabbitMQ.Stream.Client 有提供 `ILogger`，可以使用 `Microsoft.Extensions.Logging` 來實作
3. RabbitMQ.Stream.Client 有提供 `StreamConsumer` 來實作 consumer，但是沒有提供 `StreamProducer`，因此需要自己實作

Consumer 的 offset type 有幾種選擇：

1. OffsetTypeFirst

    > 從第一個可用 offset 開始。如果 stream 未被截斷，則這表示 stream 的開頭（offset 0）

2. OffsetTypeLast

    > 從 stream 結束開始，立即返回最後一個訊息（前提是 stream 中有資料）

3. OffsetTypeNext

    > 從下一個要寫入的 offset 開始。與 OffsetTypeLast 相反，如果沒有人發佈到 stream，則使用 OffsetTypeNext 進行消費將不會傳回任何內容。當訊息發佈到 stream 時，將開始向消費者發送訊息

4. OffsetTypeOffset(offset)

    > 從指定的 offset 開始。 0 表示從 stream 的開頭開始消費（第一則訊息）。也可以指定任何數字，例如在應用程式的前一個版本中停止的偏移量

5. OffsetTypeTimestamp(timestamp)

    > 從指定時間戳記之後儲存的訊息開始，參數允許使用 `DateTime`' 或 `DateTimeOffset` 與 `unix timestamp`(精準度至 `millisecond`)

我自己測試下來，除了 `OffsetTypeTimestamp` 外，其他設定都符合預期，`OffsetTypeTimestamp` 會出現時間對不上的狀況：我在 produce 時在 message 中加上 `DateTimeOffset.UtcNow.ToUnixTimeMilliseconds`，接透過 `OffsetTypeFirst` 取得所有 message 並將 `MessageContext` 中的 timestamp 輸出，結果出現寫入 message 的 timestamp 與 `MessageContext` 紀錄的有大幅落差；以其中一筆紀錄 `Received message: 0 @ 1699237873576 |0 | 1699056603023` 為例，`1699237873576` GMT (2023年11月6日Monday 02:31:13.576) 是 message 中的 timestamp，`1699056603023` (GMT 2023年11月4日Saturday 00:10:03.023) 是 `MessageContext` 紀錄的 timestamp，兩者相差 `181270553`，約 `2天21小時11分`，`MessageContext` 紀錄的資訊提早了如此多，我檢查了 RabbitMQ 的時區並無異常，目前還不清楚為什麼會出現這樣的情況，不知道是不是還有哪邊的設定影響到，所以在找到原因前，我個人是不會考慮使用 `OffsetTypeTimestamp` 的

![3imtestampmessage](https://github.com/yowko/picsbed/assets/3851540/1bae6924-fbbe-4569-9beb-f0ddfe903c91)

完整程式碼：[yowko/rabbitmq-streams-binary](https://github.com/yowko/rabbitmq-streams-binary)

## 參考資訊

1. [使用 C# 存取 RabbitMQ Streams](/csharp-rabbitmq-streams)
2. [RabbitMQ Streams and Replay Features, Part 1: When to Use RabbitMQ Streams](https://www.cloudamqp.com/blog/rabbitmq-streams-and-replay-features-part-1-when-to-use-rabbitmq-streams.html)
3. [RabbitMQ Streams and replay features, Part 2: How to work with RabbitMQ Streams](https://www.cloudamqp.com/blog/rabbitmq-streams-and-replay-features-part-2-how-to-work-with-rabbitmq-streams.html)
4. [RabbitMQ Stream .Net Client](https://rabbitmq.github.io/rabbitmq-stream-dotnet-client/stable/htmlsingle/index.html)
5. [docs/Documentation/GettingStarted.cs](https://github.com/rabbitmq/rabbitmq-stream-dotnet-client/blob/main/docs/Documentation/GettingStarted.cs)
6. [yowko/rabbitmq-streams-binary](https://github.com/yowko/rabbitmq-streams-binary)
