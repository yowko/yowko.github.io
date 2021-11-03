---
title: "調整 Kafka 中 Topic 的 Partition 數量"
date: 2019-06-07T23:30:00+08:00
lastmod: 2021-11-03T23:30:31+08:00
draft: false
tags: ["Kafka","csharp"]
slug: "change-kafka-topic-partition"
---

## 調整 Kafka 中 Topic 的 Partition 數量

最近積極在針對 Kafka 做些設定調整與效能測試，其中也包含了 Topic 的 Partition 數量產生的影響，也就常常需要調整 Topic 中的 Partition 數量來檢視對效能的影響，簡單紀錄一下我自己常用的做法備忘

## 基本環境說明

1. macOS Mojave 10.14.5
2. Docker Engine - Community 18.09.2
3. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
4. Kafka 2.2.1
5. NuGet Package

    - Confluent.Kafka 1.0.1.1

## 調整方式

1. 使用官方 shell

    > 這是過去我一直以來的做法

    - 基本語法

        ```bash
        kafka-topics.sh --alter --zookeeper {kafka 位置} --topic {topic 名稱} --partitions {partition 個數}
        ```

    - 實際範例

         ```bash
        kafka-topics.sh --alter --zookeeper localhost:2181 --topic my-topic --partitions 3
        ```

2. 使用 `Confluent.Kafka`

    > 這是最近的發現 `1.0.0` (2019/4/26) 版本加入的新功能

    ```cs
    const string bootstrapServers = "127.0.0.1:9092";
    const string topicName = "TestPartition";

    using (var adminClient =
        new AdminClientBuilder(config: new AdminClientConfig {BootstrapServers = bootstrapServers}).Build())
    {
        try
        {
            //調整 partition
           await adminClient.CreatePartitionsAsync(new List<PartitionsSpecification>
           {
               //指定 Topic 及目標 partition 數
               new PartitionsSpecification { Topic = topicName, IncreaseTo = 3 }
           });
        }
        catch (CreateTopicsException e)
        {
            Console.WriteLine(
                value:
                $"An error occured creating topic {e.Results[index: 0].Topic}: {e.Results[index: 0].Error.Reason}");
        }
    }
    ```

## 心得

不論是 shell 或是 `Confluent.Kafka`，partition 數量都只能增加不能減少，就算是指定數量與當下數量一致也會出現錯誤訊息，使用上需要留意

可以使用 `./kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-topic` 先確認目前設定值

## 參考資訊

1. [Adding Partitions to a Topic in Apache Kafka](https://www.allprogrammingtutorials.com/tutorials/adding-partitions-to-topic-in-apache-kafka.php)
