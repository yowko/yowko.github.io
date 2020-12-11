---
title: "使用 .Net client 連線至 Apache Kafka 收發訊息"
date: 2017-05-25T01:00:00+08:00
lastmod: 2020-12-11T01:00:27+08:00
draft: false
tags: ["Kafka","C#"]
slug: "kafka-client-produce-consume"
aliases:
    - /2017/05/kafka-client-produce-consume.html
---
# 使用 .Net client 連線至 Apache Kafka 收發訊息

<span style="color:red">.NET Core 用法可以參考 [讓 Kafka 達成 Broadcast 效果](/kafka-broadcast-assign/)</span>


繼之前紀錄 [如何在 Windows OS 安裝 Apache Kafka](//blog.yowko.com/2017/03/windows-os-apache-kafka.html) 到現在默默地過了兩個月XD，直到最近才有時間可以再開始進行 MQ 相關功能比較，經過初步比較後，目前打算以 RabbitMQ 與 Apache Kafka 兩個 MQ 系統為主要選項來做比較，之前文章 [使用 .Net client 連線至 RabbitMQ 收發訊息](//blog.yowko.com/2017/05/rabbitmq-client-send-consume.html) 介紹了使用 .Net Client 連線 RabbitMQ 進行基本發送訊息及接收訊息的程式碼，今天就來看看 .Net 如何連線 Apache Kafka 進行收發訊息

身為 .Net 工程師，雖然想多熟悉 linux，但終究沒有那麼容易，今天的 demo 還是會以 Windows 上的 Kafka 當做範例，如果對 Windows 上安裝 Apache Kafka 有興趣的可以參考 [如何在 Windows OS 安裝 Apache Kafka](//blog.yowko.com/2017/03/windows-os-apache-kafka.html)

## 安裝 .Net Client

*   ~~[Microsoft/CSharpClient-for-Kafka](https://github.com/Microsoft/CSharpClient-for-Kafka)~~

    > Kafka .Net Client 在 NuGet 上有好幾套件，其實並不好選擇，本來想使用 Microsoft 出的 [CSharpClient-for-Kafka](https://github.com/Microsoft/CSharpClient-for-Kafka)，但專案說明出現：

    > 只支援到 Kafka 0.8，直到 Kafka 1.0 才會重新包裝 library，如果是 Kafaka 0.9 之後版本請使用 [confluent-kafka-dotnet](https://github.com/confluentinc/confluent-kafka-dotnet)

    ![1msnote](https://cloud.githubusercontent.com/assets/3851540/26392985/fa28a29e-409a-11e7-8a38-cf7212573d49.png)

*   [confluentinc/confluent-kafka-dotnet](https://github.com/confluentinc/confluent-kafka-dotnet)

    *   使用 Package Manager Console

        ```
        Install-Package Confluent.Kafka -Version 0.9.5
        ```

        >官方建議的安裝語法有指定版本為 0.9.5 我看最新版本就是 0.9.5 ，不指定還是會安裝 0.9.5

    *   使用 NuGet Package Explorer

        1.  專案(project) 上按右鍵 --> Manage NuGet Packages...

            ![2nugetmanage](https://cloud.githubusercontent.com/assets/3851540/26392986/fa2e19c2-409a-11e7-9c96-2bd621b9b87a.png)

        2.  Browse --> 搜尋 `confluent.kafka` --> 點選 `Confluent.Kafka` --> Install

            ![3nugetinstall](https://cloud.githubusercontent.com/assets/3851540/26392984/fa269bb6-409a-11e7-8099-27d038d6ad3a.png)

## 接收訊息

> 使用 confluent-kafka-dotnet 的[官方範例](https://github.com/confluentinc/confluent-kafka-dotnet/tree/master/examples/SimpleConsumer)來建立 consumer

*   範例程式

    ```cs
    void Main()
    {
        //指定 kafka 所在位置及 port
        string brokerList = "localhost:9092";
        //指定要監聽的 topic，可以監聽多個
        var topics = new List<string>() { "Yowkotest" };
        //這個 group.id 沒什麼作用，可以隨便給，將 kafka 位置設定給 config
        var config = new Dictionary<string, object>
        {
            { "group.id", "yowko" },
            { "bootstrap.servers", brokerList }
        };
        //將 config 餵給 consumer
        using (var consumer = new Consumer<Null, string>(config, null, new StringDeserializer(Encoding.UTF8)))
        {
            consumer.Assign(new List<TopicPartitionOffset> { new TopicPartitionOffset(topics.First(), 0, 0) });
            //持續監聽
            while (true)
            {
                Message<Null, string> msg;
                //接受訊息
                if (consumer.Consume(out msg))
                {
                    Console.WriteLine($"Topic: {msg.Topic} Partition: {msg.Partition} Offset: {msg.Offset} {msg.Value}");
                }
            }
        }
    }
    ```

*   注意事項
    *   只會接受 producer 在 consumer 啟動後所發送的訊息
    *   多個 consumer 都會收到同樣訊息，不是分配接受端



## 發送訊息

*   範例程式

    ```cs
    void Main()
    {
        //指定 kafka 所在位置及 port
        string brokerList = "localhost:9092";
        //指定發送的 topic
        string topicName = "Yowkotest";
        //將 kafka 位置設定給 config
        var config = new Dictionary<string, object> { { "bootstrap.servers", brokerList } };
        //將 config 餵給 producer
        using (var producer = new Producer<Null, string>(config, null, new StringSerializer(Encoding.UTF8)))
        {
            Console.WriteLine($"{producer.Name} producing on {topicName}. q to exit.");
            string text;
            //持續等待 user 輸入
            while ((text = Console.ReadLine()) != "q")
            {
                // 非同步發送訊息至 broker
                var deliveryReport = producer.ProduceAsync(topicName, null, text);
                //發送成功後輸出訊息
                deliveryReport.ContinueWith(task =>
                    {
                        Console.WriteLine($"Partition: {task.Result.Partition}, Offset: {task.Result.Offset}, Message: {text}");
                    });
            }
            //將 producer request 保留至 disk，確保資料不會遺失
            producer.Flush();
        }
    }
    ```

*   注意事項

    *   flush 動作不一定要執行，建議只針對重要訊息執行即可，會影響效能
    *   如果會關閉 producer ，建議執行避免有未完成的 request 遺失



## 其他選項

*   使用其他 .Net Client -  [ah-/rdkafka-dotnet](https://github.com/ah-/rdkafka-dotnet)
*   範例程式

    ```cs
    void Main()
    {
        // 這個 GroupId 沒什麼作用，可以隨便給
        var config = new Config() { GroupId = "example-csharp-consumer" };
        //將 config 跟 kafaka host 跟 port 指定跟 consumer
        using (var consumer = new EventConsumer(config, "127.0.0.1:9092"))
        {
            //如果收到訊息時的行為
            consumer.OnMessage += (obj, msg) =>
            {
                string text = Encoding.UTF8.GetString(msg.Payload, 0, msg.Payload.Length);
                //取得訊息後輸出
                Console.WriteLine($"Topic: {msg.Topic} Partition: {msg.Partition} Offset: {msg.Offset} {text}");
            };
            // consumer 訂閱 topic
            consumer.Subscribe((new[] { "Yowkotest" }).ToList());
            // 開始監聽
            consumer.Start();
            Console.WriteLine("Started consumer, press enter to stop consuming");
            Console.ReadLine();
        }
    }
    ```

*   注意事項
    *   只有沒收過的訊息，啟動監聽後會全部收下來
    *   一樣是非分配訊息接受，訊息會被後起的 consumer 收走



## 心得

Kafka 在 .Net 上的資源相對於 RabbitMQ 是比較少的，設定上也較煩瑣，周邊配套功能或是工具支援都較少，不像 RabbitMQ 成熟，可能因為發展時間造成的。

功能跟預期(可以自動分流 consumer)不同，我想應該是設定面問題，這個再找時間進行研究跟調整，有新的心得再跟大家分享

<span style="color:red">.NET Core 用法可以參考 [讓 Kafka 達成 Broadcast 效果](/kafka-broadcast-assign/)</span>

# 參考資訊

1.  [如何在 Windows OS 安裝 Apache Kafka](//blog.yowko.com/2017/03/windows-os-apache-kafka.html)
2.  [使用 .Net client 連線至 RabbitMQ 收發訊息](//blog.yowko.com/2017/05/rabbitmq-client-send-consume.html)
3.  [Microsoft/CSharpClient-for-Kafka](https://github.com/Microsoft/CSharpClient-for-Kafka)
4.  [confluentinc/confluent-kafka-dotnet](https://github.com/confluentinc/confluent-kafka-dotnet)
5.  [ah-/rdkafka-dotnet](https://github.com/ah-/rdkafka-dotnet)
