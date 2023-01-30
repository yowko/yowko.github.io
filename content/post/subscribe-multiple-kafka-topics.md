---
title: "同時訂閱多個 Kafka topic"
date: 2023-01-18T00:30:00+08:00
lastmod: 2023-01-18T00:30:31+08:00
draft: false
tags: ["kafka","csharp"]
slug: "subscribe-multiple-kafka-topics"
---

## 同時訂閱多個 Kafka topic

隨著產品的發展，團隊所建立的 application 也逐漸變多，不同 application 的溝通也變得複雜，為了避免直接相依，一部份的功能是透 kafka 來交換資料，但如果不加以管理，可以想見 kafka topic 的數量與 message 的大小、發送頻率很快就會失控，所以團隊就想要針對 kafka 的 topic 命名加上一些規範：

1. [原生限制] 長度不得超過 `255` 字元
2. [原生限制] 只允許 `a-z`, `A-Z`, `0-9`, `. (dot)`, `_ (underscore)` 與 `- (dash)`
3. [團隊建議] 命名格式為：`{發送端 application name}-{用途}` ex：`auth-log`

前兩項因為是 kafka 原生的限制，沒什麼討論空間，但第三點的團隊建議就有不同聲音了。希望以 `發送端 application name` 而不是接收端 application name 的原因是希望可以迅速定位到資料源頭也避免 topic 遭到誤用；以上面 `auth-log` 為例，如果改成接收端 (filebeat) 來命名：`filebeat-log`，這時候有個新 application 也將 log 接入 `filebeat-log` 可是格式不符，造成 log 爬不出來，這時候就需要逐一排除到底是哪個 application 造成的問題，也可能造成到其他沒問題 application 無法輸出 log

想要使用接收端 (filebeat) 來命名主要是因為需要同時接收多個 application 提供的資料，建立多個 consumer 很麻煩，但其實 kafka 是支援同時訂閱多個 topic 的，今天就快速紀錄一下使用方式

## 基本環境說明

1. macOS Ventura 13.1
2. .NET SDK 6.0.400
3. NuGet packages
    - Confluent.Kafka 1.9.3
4. docker images
    - quay.io/strimzi/kafka:latest-kafka-3.3.1-amd64
5. kafka 建立方式

    > 使用 kraft 算法而不用相依於 zookeeper 的方式來建立 kafka，詳細內容可以參考 [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper/)

    - server.properties

        ```config
        process.roles=broker,controller
        node.id=1
        offsets.topic.replication.factor=1
        controller.quorum.voters=1@kafka:9093
        listeners=PLAINTEXT://:9092,CONTROLLER://:9093
        advertised.listeners=PLAINTEXT://kafka:9092
        controller.listener.names=CONTROLLER
        listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
        log.dirs=kafka_data
        ```

    - docker-compose.yaml

        ```yaml
        version: '3'
        services:
          kafka:
            container_name: kafka
            user: root
            image: quay.io/strimzi/kafka:latest-kafka-3.3.1-amd64
            volumes:
              - "./server.properties:/opt/kafka/config/kraft/server.properties"
            command:
              [
                "sh",
                "-c",
                "bin/kafka-storage.sh format -t $$(bin/kafka-storage.sh random-uuid) -c /opt/kafka/config/kraft/server.properties && bin/kafka-server-start.sh /opt/kafka/config/kraft/server.properties"
              ]
            ports:
              - "9092:9092"
        ```

    - topic 建立

        ```bash
        ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic t1 && ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic t2
        ```

## 設定方式

1. C# code：進行 topic subscribe，傳入多個 topic

    ```cs
    class Program
    {
        static void Main(string[] args)
        {
            var conf = new ConsumerConfig
            { 
                GroupId = "yowkotest",
                BootstrapServers = "localhost:9092",
                AutoOffsetReset = AutoOffsetReset.Earliest
            };

            using var c = new ConsumerBuilder<Ignore, string>(conf).Build();
            //進行 topic subscribe，使用多個 topic
            c.Subscribe(new []{"t1","t2"});
            
            CancellationTokenSource cts = new CancellationTokenSource();
            Console.CancelKeyPress += (_, e) => {
                e.Cancel = true; // prevent the process from terminating.
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
        }
    }
    ```

2. 實際效果

    ![_output_1result](https://user-images.githubusercontent.com/3851540/213143842-4c7f05ee-2d69-4608-b687-c712a3187785.png)

## 心得

看得出來 code 修改的部份很小： `c.Subscribe("topic");` --> `c.Subscribe(new []{"t1","t2"});`，甚至以後預設都改用傳入 array 的方式來 `subscribe` 也行

這樣的筆記自己都覺得太水，但剛好趁這個機會驗證一下順便紀錄，大家討論時可以比較有信心

## 參考資訊

1. [Producing messages on Kafka topics](https://www.ibm.com/docs/en/integration-bus/10.0?topic=bus-producing-messages-kafka-topics)
2. [Kafka: Consume Messages From Multiple Queues](https://medium.com/@maverick11/kafka-consume-messages-from-multiple-queues-bdf0e07329b1)
3. [使用 Docker 啟動不依賴 ZooKeeper 的 Kafka](/docker-kafka-without-zookeeper/)
