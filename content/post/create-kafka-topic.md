---
title: "建立 Kafka 的 Topic"
date: 2019-06-07T21:30:00+08:00
lastmod: 2020-12-23T21:30:31+08:00
draft: false
tags: ["Kafka","csharp"]
slug: "create-kafka-topic"
---

## 建立 Kafka 的 Topic

最近系統效能瓶頸主要落在在 Kafka 上，所以經常需要對 Kafka 做些設定調整與效能測試，而在那之前首先就是要建立 Topic，過去都是透過 kafka 預設值來自動建立，但這樣一來就無法設定一些細微參數，遇到想要微調設定就得要手動建立，剛好 google 到有可以透過程式來建立 Topic 順手紀錄一下不同用法

## 基本環境說明

1. macOS Mojave 10.14.5
2. Docker Engine - Community 18.09.2
3. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
4. Kafka 2.2.1
5. NuGet Package

    - Confluent.Kafka 1.0.1.1

## 建立方式

1. 預設連線時建立

    > Kafka 的預設 `auto.create.topics.enable = true`，加上在 Producer 或是 Consumer 連線至 Kafka 時都需要指定 Topic，因此建立連線當下就會使用預設值建立對應的 Topic

2. 使用官方 shell

    - 基本語法

        ```bash
        kafka-topics.sh --create --bootstrap-server {kafka 位置} --config {configName=configValue} --replication-factor {個數} --partitions {個數} --topic {Topic 名稱}
        ```

    - 實際範例

        ```bash
        kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
        ```

3. 透過 Confluent.Kafka

    > 這是 `1.0.0` 版本才加上的功能，也是 2019/4/26 才推出的

    ```cs
    const string bootstrapServers = "127.0.0.1:9092";
    const string topicName = "testtopic";

    using (var adminClient =
        new AdminClientBuilder(config: new AdminClientConfig {BootstrapServers = bootstrapServers}).Build())
    {
        try
        {
            await adminClient.CreateTopicsAsync(new TopicSpecification[]
            {
                new TopicSpecification {
                    Name = topicName,
                    ReplicationFactor = 1, //副本數
                    NumPartitions = 1,// partition 數量
                    // config 依實際需要設定
                    //Configs = new Dictionary<string, string>()
                    //{
                    //    {"max.message.bytes","1000012"}
                    //}}
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

過去因為無法透過程式來建立或是設定 topic，所以常常需要先手動指定參數並建立 topic 後才能滿足當下目的，不過最近重新 google 卻發現原來已經加上這個功能了  有種賺到的感覺XD

雖然我習慣在實作相似功能時 間隔三個月以上就會 google 看看有沒有其他更新、更好、更漂亮的做法，但不免還是有漏網之漁呀，常常沒辦法 catch 到最新的資訊，我猜自文不好讓我總是慣性略過英文技術介紹影響還是挺大的 QQ

## 參考資訊

1. [Step 3: Create a topic](https://kafka.apache.org/documentation/#quickstart_createtopic)
2. [How to create a Kafka Topic using Confluent.Kafka .Net Client](https://stackoverflow.com/a/55849553)
3. [Adding Partitions to a Topic in Apache Kafka](https://www.allprogrammingtutorials.com/tutorials/adding-partitions-to-topic-in-apache-kafka.php)
