---
title: "讓 Kafka 達成 Broadcast 效果"
date: 2020-04-05T21:30:00+08:00
lastmod: 2020-04-05T21:30:31+08:00
draft: false
tags: ["dotnet core","Kafka"]
slug: "kafka-broadcast-assign"
---

## 讓 Kafka 達成 Broadcast 效果

Kafka 在處理訊息上主要是透過 Consumer GroupId + Topic + Partition 做為 Unique 的派送訊息基準，預設(未指定 Partition)下會由 Kafka 自行決定 Partition

在上述的原則下，如果 message 只需要被某個 consumer 處理大致上都沒有問題，而一旦情境轉變為 `通知所有 consumer` (message 需要被每個 consumer 處理) 上述原則就變得限制太多，常見的做法就是將每個 consumer 使用的 Consumer GroupId 錯開，但這麼一來就會讓 Consumer GroupId 數量大增，大大不利於管理以及監控

這次專案也遇到類似問題，所以簡單紀錄一下可能的解決方式之一

## 基本環境說明

1. macOS Catalina 10.15.4
2. .NET Core SDK 3.1.102
3. docker deskop community 2.2.0.4(43472)
4. wurstmeister/kafka:2.12-2.4.0
5. wurstmeister/zookeeper:3.4.6
6. NuGet packages

    - Confluent.Kafka 1.4.0

## 修改前 (使用 `Subscribe`)

1. Producer

    ```cs
    var config = new ProducerConfig { BootstrapServers = "localhost:9092" };

    using var p = new ProducerBuilder<Null, string>(config).Build();
    try
    {
        var dr = await p.ProduceAsync("test-topic", new Message<Null, string> { Value="test" });
        Console.WriteLine($"Delivered '{dr.Value}' to '{dr.TopicPartitionOffset}'");
    }
    catch (ProduceException<Null, string> e)
    {
        Console.WriteLine($"Delivery failed: {e.Error.Reason}");
    }
    ```

2. Consumer

    > 兩個 consumer 程式碼大致相同，只在處理 message 時加上不同 consumer 的log 而已

    - Consumer 1

        ```cs
        var conf = new ConsumerConfig
        {
            GroupId = "test-consumer-group",
            BootstrapServers = "localhost:9092",
            AutoOffsetReset = AutoOffsetReset.Latest
        };

        using var c = new ConsumerBuilder<Ignore, string>(conf).Build();

        c.Subscribe("test-topic");

        var cts = new CancellationTokenSource();
        Console.CancelKeyPress += (_, e) => {
            e.Cancel = true;
            cts.Cancel();
        };

        try
        {
            while (true)
            {
                try
                {
                    var cr = c.Consume(cts.Token);
                    Console.WriteLine($"[Consumer1]:Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
                }
                catch (ConsumeException e)
                {
                    Console.WriteLine($"Error occured: {e.Error.Reason}");
                }
            }
        }
        catch (OperationCanceledException)
        {
            c.Close();
        }
        ```

    - Consumer 2

        ```cs
        var conf = new ConsumerConfig
        {
            GroupId = "test-consumer-group",
            BootstrapServers = "localhost:9092",
            AutoOffsetReset = AutoOffsetReset.Latest
        };

        using var c = new ConsumerBuilder<Ignore, string>(conf).Build();

        c.Subscribe("test-topic");

        var cts = new CancellationTokenSource();
        Console.CancelKeyPress += (_, e) => {
            e.Cancel = true;
            cts.Cancel();
        };

        try
        {
            while (true)
            {
                try
                {
                    var cr = c.Consume(cts.Token);
                    Console.WriteLine($"[Consumer2]:Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
                }
                catch (ConsumeException e)
                {
                    Console.WriteLine($"Error occured: {e.Error.Reason}");
                }
            }
        }
        catch (OperationCanceledException)
        {
            c.Close();
        }
        ```

3. 實際效果

    > 多個 consumer 在相同 group id 以及 topic 下，僅一個 consumer 會收到 message

    - Consumer 1

        ![1consumer-before](https://user-images.githubusercontent.com/3851540/78500850-83959800-778b-11ea-9e6d-016203659772.png)

    - Consumer 2

        ![1consumer-before](https://user-images.githubusercontent.com/3851540/78500853-84c6c500-778b-11ea-9bc7-cb876ec48df9.png)

## 修改後 (使用 `Assign`)

將 `Subscribe` 改用 `Assign` (手動指定 partition id) 後會解除原本 Consumer GroupId + Topic + Partition 做為 Unique 派送訊息基準的限制

1. 修改方式

    > 改用 `Assign` 並指定 topic 以及 partition id

    - Consumer 1

        ```cs
        var conf = new ConsumerConfig
        {
            GroupId = "test-consumer-group",
            BootstrapServers = "localhost:9092",
            AutoOffsetReset = AutoOffsetReset.Latest
        };

        using var c = new ConsumerBuilder<Ignore, string>(conf).Build();

        //c.Subscribe("test-topic");
        c.Assign(new TopicPartition("test-topic",0));

        var cts = new CancellationTokenSource();
        Console.CancelKeyPress += (_, e) => {
            e.Cancel = true;
            cts.Cancel();
        };

        try
        {
            while (true)
            {
                try
                {
                    var cr = c.Consume(cts.Token);
                    Console.WriteLine($"[Consumer1]:Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
                }
                catch (ConsumeException e)
                {
                    Console.WriteLine($"Error occured: {e.Error.Reason}");
                }
            }
        }
        catch (OperationCanceledException)
        {
            c.Close();
        }
        ```

    - Consumer 2

        ```cs
        var conf = new ConsumerConfig
        {
            GroupId = "test-consumer-group",
            BootstrapServers = "localhost:9092",
            AutoOffsetReset = AutoOffsetReset.Latest
        };

        using var c = new ConsumerBuilder<Ignore, string>(conf).Build();

        //c.Subscribe("test-topic");
        c.Assign(new TopicPartition("test-topic",0));

        var cts = new CancellationTokenSource();
        Console.CancelKeyPress += (_, e) => {
            e.Cancel = true;
            cts.Cancel();
        };

        try
        {
            while (true)
            {
                try
                {
                    var cr = c.Consume(cts.Token);
                    Console.WriteLine($"[Consumer2]:Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
                }
                catch (ConsumeException e)
                {
                    Console.WriteLine($"Error occured: {e.Error.Reason}");
                }
            }
        }
        catch (OperationCanceledException)
        {
            c.Close();
        }
        ```

2. 實際效果

    > 相同 consumer group id 與 topic，也能讓每個 consumer 都收到訊息

    - Consumer 1

        ![3consumer1-after](https://user-images.githubusercontent.com/3851540/78500854-85f7f200-778b-11ea-9b96-ff5819c004ec.png)

    - Consumer 2

        ![4consumer2-after](https://user-images.githubusercontent.com/3851540/78500855-86908880-778b-11ea-828a-39fd50777979.png)

## 心得

使用 `Assign` 需要手動指定 Partition，而這個動作牽涉到 Kafka 自動分配 Partition 以及 rebalancing 機制，如果應用不當可能會造成 Kafka 在處理 offset 上的混亂，使用上要特別留意不要在同個 topic 上混用 `Assign` 與 `Subscribe`

## 參考資訊

1. [confluentinc/confluent-kafka-dotnet](https://github.com/confluentinc/confluent-kafka-dotnet)
2. [Don't Use Apache Kafka Consumer Groups the Wrong Way!](https://dzone.com/articles/dont-use-apache-kafka-consumer-groups-the-wrong-wa)
