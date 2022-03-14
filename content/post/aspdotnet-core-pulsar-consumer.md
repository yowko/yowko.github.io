---
title: "在 ASP.NET Core 中從 Apache Pulsar 接收訊息"
date: 2022-03-10T00:30:00+08:00
lastmod: 2022-03-10T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp","Pulsar"]
slug: "aspdotnet-core-pulsar-consumer"
---

## 在 ASP.NET Core 中從 Apache Pulsar 接收訊息

Apache Pulsar 常常被拿來與 Kafka 做比較，孰優孰劣常常也是各自擁護者爭相討論的內容，以下條列幾項選擇 Pulsar 的正面意見

1. 同時支援即時訊息與訊息佇列
2. 支援 partition 但可以選擇不用
3. 分散式 log
4. 無狀態
5. 跨地域性複製
6. 較高吞吐量的同時保持較低的延遲
7. 分層儲存與多租戶
8. 可以簡易與 kafka 、 rabbitmq 整合

其實列了一堆，我個人是不太在意啦，當然有些內容是有打中 kafka 的部份缺點，不過我會想嘗試的主因是有效能測試比較 - [Benchmarking Pulsar and Kafka - A More Accurate Perspective on Pulsar's Performance](https://streamnative.io/blog/tech/2020-11-09-benchmark-pulsar-kafka-performance/) ：Pulsar 在大部份情境的性能都優於 kafka，加上 kafka 在大資料量下可能會出現延遲，雖然還沒實際遇到問題，但我還是想先預先增加一些技術 solution，以備不時之需

之前筆記 [在 ASP.NET Core 中發送訊息至 Apache Pulsar](/aspdotnet-core-pulsar-producer) 紀錄到如何在 ASP.NET Core 中如何發送訊息，今天要紀錄透過 Consumer 接受訊息的使用方式

如果想要透過 Reader 來獲取訊息可以參考 [在 ASP.NET Core 中從 Apache Pulsar 接收訊息 (Reader)](/aspdotnet-core-pulsar-reader)

Reader 與 Consumer 的差異在於

1. Reader 可以指定開始處理 message 的位置，而 Consumer 都是從可用、未確認且最新的 message 開始處理
2. Reader 不需 ack 也不保留相關資料

## 基本環境說明

1. macOS Monterey 12.2.1
2. docker desktop 4.2.0(70708)
3. docker images

    - apachepulsar/pulsar:2.9.1

4. .NET SDK 6.0.200
5. NuGet packages

    - DotPulsar 2.2.0
    - Pulsar.Client 2.10.0

6. 使用 docker 啟動 pulsar

    ```bash
    docker run -d -p 6650:6650 -p 8080:8080 --name pulsar apachepulsar/pulsar:latest bin/pulsar standalone
    ```

    - `6650` 是 broker service port
    - `8080` 是 web service port

## 使用方式

1. DotPulsar

    - 建立 consumer BackgroundService

        > 下列兩個方式的差異只在於 consumer 的建立方式

        - 使用 builder

            ```cs
            public class ConsumerBuilderService:BackgroundService
            {
                private const string Topic = "persistent://public/default/yowkotest";
                private const string SubscriptionName = "YowkoSubscription";
                private readonly IPulsarClient _pulsarClient;
            
            
                public ConsumerBuilderService(IPulsarClient pulsarClient)
                {
                    _pulsarClient = pulsarClient;   
                }
            
                protected override async Task ExecuteAsync(CancellationToken stoppingToken)
                {
                    while (!stoppingToken.IsCancellationRequested)
                    {
                        await using var pulsarConsumer = _pulsarClient.NewConsumer(Schema.String)
                            .SubscriptionName(SubscriptionName)
                            .Topic(Topic)
                            .Create();
                        
                        await foreach (var message in pulsarConsumer.Messages(stoppingToken))
                        {
                            Console.WriteLine($"Received: {message.Value()}");
                            await pulsarConsumer.Acknowledge(message, stoppingToken);
                            //await pulsarConsumer.AcknowledgeCumulative(message, cancellationToken);
                        }
                        
                        await Task.Delay(TimeSpan.FromSeconds(3), stoppingToken);
                    }
                }
            }
            ```

        - 不使用 builder

            ```cs
            public class ConsumerBuilderService : BackgroundService
            {
                private const string Topic = "persistent://public/default/yowkotest";
                private const string SubscriptionName = "YowkoSubscription";
                private readonly IPulsarClient _pulsarClient;
            
            
                public ConsumerBuilderService(IPulsarClient pulsarClient)
                {
                    _pulsarClient = pulsarClient;
                }
            
                protected override async Task ExecuteAsync(CancellationToken stoppingToken)
                {
                    while (!stoppingToken.IsCancellationRequested)
                    {
                        var options = new ConsumerOptions<string>(SubscriptionName,Topic,Schema.String);
                        await using var pulsarConsumer = _pulsarClient.CreateConsumer(options);

                        await foreach (var message in pulsarConsumer.Messages(stoppingToken))
                        {
                            Console.WriteLine($"Received: {message.Value()}");
                            // 手動 ack
                            await pulsarConsumer.Acknowledge(message, stoppingToken);
                            //await pulsarConsumer.AcknowledgeCumulative(message, cancellationToken);
                        }

                        await Task.Delay(TimeSpan.FromSeconds(3), stoppingToken);
                    }
                }
            }
            ```

    - Program.cs : 註冊 Pulsar client 與 HostedService

        ```cs
        // 建立 Pulsar 連線 client，可以透過 `ServiceUrl` 設定 Pulsar 位址(預設： `pulsar://localhost:6650`)、`RetryInterval` 設定重試或是重連的間隔秒數 (預設值：3s)
        var pulsarClient = PulsarClient
            .Builder()
            .ServiceUrl(new Uri("pulsar://localhost:6650"))
            .RetryInterval(TimeSpan.FromSeconds(3))
            .Build();
        
        builder.Services.AddSingleton(pulsarClient);

        builder.Services.AddHostedService<ConsumerBuilderService>();
        //builder.Services.AddHostedService<ConsumerWithoutBuilderService>();
        ```

2. Pulsar.Client

    - 建立 consumer BackgroundService

        ```cs
        public class ReaderService : BackgroundService
        {
            private const string Topic = "yowkotest";
            private readonly PulsarClient _pulsarClient;
        
        
            public ReaderService(PulsarClient pulsarClient)
            {
                _pulsarClient = pulsarClient;
            }
        
            protected override async Task ExecuteAsync(CancellationToken stoppingToken)
            {
                while (!stoppingToken.IsCancellationRequested)
                {
                    await using var pulsarReader = await _pulsarClient.NewReader(Schema.STRING(Encoding.UTF8))
                        .Topic(Topic)
                        .StartMessageId(MessageId.Earliest)
                        .CreateAsync();
        
                    var message = await pulsarReader.ReadNextAsync(stoppingToken);
                    Console.WriteLine($"Received: {message.GetValue()}");
                }
            }
        }
        ```

    - Program.cs : 註冊 Pulsar client 與 HostedService

        ```cs
        const string serviceUrl = "pulsar://localhost:6650";
        const string topicName = "yowkotest";
        
        var client = await new PulsarClientBuilder()
            .ServiceUrl(serviceUrl)
            .BuildAsync();
        
        builder.Services.AddSingleton(client);
        builder.Services.AddHostedService<ConsumerService>();
        ```

## 心得

- 關於 DotPulsar

    1. 官網文件與 GitHub 的 wiki 不是即時的

        > library 的 api 已更新，官網上的教學未更新，照著做一定沒辦法用

        - 沒有指定 message 類型

            ![1DotPulsarmissingtmessage](https://user-images.githubusercontent.com/3851540/157833323-30205bf9-70ed-42ea-bfc3-1bde73d65e68.png)

    2. 方法命名不好： async 方法，名稱未以 async 結尾

        ![2DotPulsarmethodname](https://user-images.githubusercontent.com/3851540/157833337-e6981e56-9e18-4b3d-9731-f8a530168d5c.png)

    3. 官網上手動 ack 的程式碼錯誤

        ![3DotPulsarackwrongcode](https://user-images.githubusercontent.com/3851540/157833339-d1fb1437-72cf-45b9-90bf-d5d8c9dba931.png)

    4. 這個可能是我自己不懂造成的：沒有說明兩種 ack 的差異

- 關於 Pulsar.Client

    1. sample code 沒有指定 message 類型

        ![5PulsarClientmissingtmessage](https://user-images.githubusercontent.com/3851540/157833347-a3136b1f-7740-46af-8673-a1d67d82e2e6.png)

    2. GitHub 上的 sample 只看到一筆筆收資料的方式，不知道大資料效能如何
    3. NuGet 上的下載數與 GitHub 上的 star 數都是 Pulsar.Client 多一些

完整程式碼可以參考 [yowko/PulsarTest](https://github.com/yowko/PulsarTest)

## 參考資訊

1. [Benchmarking Pulsar and Kafka - A More Accurate Perspective on Pulsar's Performance](https://streamnative.io/blog/tech/2020-11-09-benchmark-pulsar-kafka-performance/)
2. [Set up a standalone Pulsar in Docker](https://pulsar.apache.org/docs/en/standalone-docker/)
3. [apachepulsar/pulsar](https://hub.docker.com/r/apachepulsar/pulsar)
4. [Pulsar configuration](https://pulsar.apache.org/docs/en/reference-configuration/)
5. [Pulsar C# client](https://pulsar.apache.org/docs/en/client-libraries-dotnet/)
6. [apache/pulsar-dotpulsar](https://github.com/apache/pulsar-dotpulsar)
7. [fsprojects/pulsar-client-dotnet](https://github.com/fsprojects/pulsar-client-dotnet)
8. [yowko/PulsarTest](https://github.com/yowko/PulsarTest)
9. [在 ASP.NET Core 中發送訊息至 Apache Pulsar](/aspdotnet-core-pulsar-producer)
10. [在 ASP.NET Core 中從 Apache Pulsar 接收訊息 (Reader)](/aspdotnet-core-pulsar-reader)
