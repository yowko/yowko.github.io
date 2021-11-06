---
title: "放大 kafka message size"
date: 2021-11-05T00:30:00+08:00
lastmod: 2021-11-05T00:30:31+08:00
draft: false
tags: ["csharp",".NET Core","kafka"]
slug: "kafka-max-message-size"
---

## 放大 kafka message size

今天 prod 監控噴出大量 Message size too large 的錯誤訊息，訊息內容很明確：就是傳到 kafka 的 message 太大 (預設為 `1048588` 約 1MB)，趁著這個機會紀錄一下  調整方式與使用方式

## 基本環境說明

1. CentOS 8 kernel 5.7.2
2. zookeeper 3.5.9
3. Kafka 2.6.2
4. .NET SDK 5.0.401
5. NuGet packages

    - Confluent.Kafka 0.9.4
    - Confluent.Kafka 1.8.2

6. code

    - producer

        ```cs
        var conf = new ProducerConfig
            {
                BootstrapServers = "localhost:9092"
            };

            Action<DeliveryReport<Null, string>> handler = r => 
                Console.WriteLine(!r.Error.IsError
                    ? $"Delivered message to {r.TopicPartitionOffset}"
                    : $"Delivery Error: {r.Error.Reason}");

            using var p = new ProducerBuilder<Null, string>(conf).Build();
            p.Produce("yowkotest", new Message<Null, string> { Value = GetStringFromFile("GetMatchById_18814541.json") }, handler);

            p.Flush(TimeSpan.FromSeconds(10));
        ```

    - consumer

        - 1.8.2

            ```cs
            var conf = new ConsumerConfig
            { 
                GroupId = "yowkotest",
                BootstrapServers = "localhost:9092",
                AutoOffsetReset = AutoOffsetReset.Earliest
            };

            using var c = new ConsumerBuilder<Ignore, string>(conf).Build();
            c.Subscribe("yowkotest");
            
            CancellationTokenSource cts = new CancellationTokenSource();
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
                        Console.WriteLine($"Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
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

        - 0.9.4

            ```cs
             var brokerList = "localhost:9092";
            
            var config = new Dictionary<string, object>
            {
                { "group.id", "yowkotest" },
                { "bootstrap.servers", brokerList }
            };

            using var consumer = new Consumer<Null, string>(config, null, new StringDeserializer(Encoding.UTF8));
            consumer.Assign(new List<TopicPartitionOffset> { new TopicPartitionOffset("yowkotest", 0, 0) });

            while (true)
            {
                Message<Null, string> msg;
                if (consumer.Consume(out msg))
                {
                    Console.WriteLine($"Topic: {msg.Topic} Partition: {msg.Partition} Offset: {msg.Offset} {msg.Value}");
                }
            }
            ```

## 情況描述與設定方式

1. 調整 producer config

    ```cs
    var conf = new ProducerConfig
        {
            MessageMaxBytes = 104857600 //放大為 10MB
        };
    ```

    - 未設定錯誤
        - 錯誤訊息

            ```log
            Unhandled exception. Confluent.Kafka.ProduceException`2[Confluent.Kafka.Null,System.String]: Broker: Message size too large
            at Confluent.Kafka.Producer`2.Produce(TopicPartition topicPartition, Message`2 message, Action`1 deliveryHandler)
            at Confluent.Kafka.Producer`2.Produce(String topic, Message`2 message, Action`1 deliveryHandler)
            at KafkaProducer.Program.Main(String[] args) in /Users/yowko.tsai/POCs/ForTest/KafkaProducer/Program.cs:line 32
            ```

        - 錯誤截圖

            ![1producererror](https://user-images.githubusercontent.com/3851540/140489600-7d0064ac-c645-46a2-b739-72179646ebc3.png)

2. 調整 kafka 允許的 message size

    - topic level

        - 建立新 topic

            - 語法

                ```bash
                kafka-topics.sh --bootstrap-server {kafka server}:{kafka port} --create --topic {topic name} --partitions {partition} --replication-factor {replication} --config max.message.bytes={size}
                ```

            - 範例

                ```bash
                kafka-topics.sh --bootstrap-server 192.168.80.3:9092 --create --topic yowkotest --partitions 1 --replication-factor 1 --config max.message.bytes=10485760
                ```

        - 修改既有 topic

            - 語法

                ```bash
                kafka-configs.sh --bootstrap-server {kafka server}:{kafka port} --alter --add-config max.message.bytes={size} --topic {topic name}
                ```

            - 範例

                ```bash
                kafka-configs.sh --bootstrap-server 192.168.80.3:9092 --alter --add-config max.message.bytes=10485760 --topic yowkotest
                ```

    - kafka instnce level (apply to all topic)
        - 動態修改

            - 語法

                ```bash
                kafka-configs.sh --bootstrap-server {kafka server}:{kafka port} --alter --add-config max.message.bytes={size} --entity-type brokers --entity-name {boker id}
                ```

            - 範例

                ```bash
                kafka-configs.sh --bootstrap-server 192.168.80.3:9092 --alter --add-config max.message.bytes=10485760 --entity-type brokers --entity-name 1
                ```

        - 修改 config (需要重啟 kafka instance)

            > 修改 `config/server.properties`

            ```config
            message.max.bytes=10485760
            ```

    - 未設定錯誤

        - 錯誤訊息

            ```log
            Delivery Error: Broker: Message size too large
            ```

        - 錯誤截圖

            ![2borkererror](https://user-images.githubusercontent.com/3851540/140489606-7de808dc-bd0b-4121-bbd7-94b14fce551a.png)

3. 調整 consumer config (實測下不需要調整)

    [confluent 官網](https://docs.confluent.io/platform/current/installation/configuration/broker-configs.html#brokerconfigs_message.max.bytes) consumer 版本如果小於 `0.10.2` 需要調整 consumer fetch size，但我測試 NuGet 上最舊版本 `0.9.4` 也不需要額外設定，我這才意識到 [confluent 官網](https://docs.confluent.io/platform/current/installation/configuration/broker-configs.html#brokerconfigs_message.max.bytes) 提到的 `0.10.2` 是不是 kafka 的版本而不是 confluent 的版本，不過 kafka `0.10.2` 實在是滿舊的  我沒有特別測試，如果日後有遇到再紀錄，這邊就提醒一下而已囉

## 心得

原本印象中只能調整 kafka 的 `server.properties` 並且重啟套用設定，經過今天查資料後才發現原來可以動態設定，不過我還是傾向設定在 config 上，其他人才能快速地取得設定值

## 參考資訊

1. [Send Large Messages With Kafka](https://www.baeldung.com/java-kafka-send-large-message)
