---
title: "讓特定 message 在 kafka 中可以有順序性"
date: 2020-07-25T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Kafka","csharp"]
slug: "kafka-message-immutable-sequence"
---

## 讓特定 message 在 kafka 中可以有順序性

之前專案為了保證 message 的順序性捨棄當時還在 0.8 版的 kafka 而選用 RabbitMQ，雖然 RabbitMQ 在效能數據上跟 kafka 不是同個量級水準，但已能充份滿足當時專案的效能需求；幾年後的新專案，因為有強烈的效能需求，所以改用 kafka，雖然也擔心過 message 順序性問題，幸虧同事夠力在流程面先避開可能的問題，剛好趁著最近看了不少 Kafk 的問題，順帶紀錄一下該如何讓 kafka 中的 message 有順序性

因為 kafka 的 message 順序性是基於 topic partition，所以如果 topic 僅使用到單一 partition，原則上可以假定 message 都是有順序性的，因此今天的紀錄也是為了 topic 使用到多個 partition 的

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3 (45519)
3. docker images

    - zookeeper 3.5.5
    - kafka 5.0.1

4. .NET Core SDK 3.1.301
5. NuGet packages

    - Confluent.Kafka 1.5.0

6. Kafka producer sample code

    ```cs
    var config = new ProducerConfig {BootstrapServers = "localhost:9092"};
    using var p = new ProducerBuilder<Null, string>(config).Build();
    try
    {
       for (int i = 0; i < 10; i++)
        {
            var dr = await p.ProduceAsync("test-topic", new Message<Null, string> { Value=$"test_{i}"});

            // 列出 message 存放的 topic:partition@offset
            Console.WriteLine($"Delivered '{dr.Value}' to {dr.Topic}:{dr.Partition}@{dr.TopicPartitionOffset.Offset}");
        }

    }
    catch (ProduceException<Null, string> e)
    {
        Console.WriteLine($"Delivery failed: {e.Error.Reason}");
    }
    ```

## 設定方式

1. 調整 kafka producer config

    > 使用 `Partitioner = Partitioner.Murmur2Random`

    ```cs
    var config = new ProducerConfig {
        BootstrapServers = "localhost:9092",
        Partitioner = Partitioner.Murmur2Random
        };
    ```

2. 為 kafka message 增加 key

    > 加入 key 的用途與 redis 的 hash tags 相同，kafka 會讓相同的 key 放在同個 partition 上以達成 message 順序性；以 key 為 int 為例

    ```cs
    using var p = new ProducerBuilder<int, string>(config).Build();
    try
    {
        for (int i = 0; i < 10; i++)
        {
            var dr = await p.ProduceAsync("test-topic", new Message<int, string> { Key = i,Value=$"test_{i}" });
            Console.WriteLine($"Delivered '{dr.Value}' to {dr.Topic}:{dr.Partition}@{dr.TopicPartitionOffset.Offset}");
        }
    }
    catch (ProduceException<Null, string> e)
    {
        Console.WriteLine($"Delivery failed: {e.Error.Reason}");
    }
    ```

3. 前後差異

    - 調整前：每個 message 都是隨機分配 partition

        ![1random1](https://user-images.githubusercontent.com/3851540/88482247-a940ea00-cf92-11ea-8dde-f9d3581b9b3f.png)

        ![2random2](https://user-images.githubusercontent.com/3851540/88482248-ac3bda80-cf92-11ea-953a-60115ce316ed.png)

        ![3random3](https://user-images.githubusercontent.com/3851540/88482249-acd47100-cf92-11ea-934c-2c25dff61f49.png)

    - 調整後：相同 key 會放在同個 partition

        ![4murmur1](https://user-images.githubusercontent.com/3851540/88482250-ad6d0780-cf92-11ea-9c09-b451e26a50ff.png)

        ![5murmur2](https://user-images.githubusercontent.com/3851540/88482251-ae059e00-cf92-11ea-92c6-4106c51177f3.png)

        ![6murmur3](https://user-images.githubusercontent.com/3851540/88482252-ae9e3480-cf92-11ea-855f-62be9ad39a33.png)

## 心得

我沒有特別花時間去查 kafka 在哪個版本加入以 key 做為 partitioner，但嘗試查了 partitioner 的定義，只是沒找到相關說明

整體說起來，設定算是簡單，但對於不熟悉的人，關鍵字並不好下，加上文件上也沒有特別提到，包含 partitioner 設定值的相關說明也沒有，稍嫌可惜，不過也可能是我個粗心沒看清楚而已，再請大家指教了

## 參考資訊

1. [How to do a same partitioner on C# like Java (Scala)](https://github.com/confluentinc/confluent-kafka-dotnet/issues/885#issuecomment-483254508)
