---
title: "在 ASP.NET Core 中發送訊息至 Apache Pulsar"
date: 2022-03-08T00:30:00+08:00
lastmod: 2022-03-08T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp","Pulsar"]
slug: "aspdotnet-core-pulsar-producer"
---

## 在 ASP.NET Core 中發送訊息至 Apache Pulsar

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

今天就先略過 Pulsar 的架構說明，直接從發送訊息來了解 Pulsar，先求有再求好

想了解 Consumer 接收訊息的做法可以參考 [在 ASP.NET Core 中從 Apache Pulsar 接收訊息](/aspdotnet-core-pulsar-consumer)或是透過 Reader 來獲取訊息可以參考 [在 ASP.NET Core 中從 Apache Pulsar 接收訊息 (Reader)](/aspdotnet-core-pulsar-reader)

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

    - Program.cs : 註冊 Pulsar Producer

        > 以下兩種方式都是官網上列的，我使用起來沒有差別，個人不負責任推測只是寫法上的差異

        - 使用 builder 建立

            ```cs
            // 建立 Pulsar 連線 client，可以透過 `ServiceUrl` 設定 Pulsar 位址(預設： `pulsar://localhost:6650`)、`RetryInterval` 設定重試或是重連的間隔秒數 (預設值：3s)
            var pulsarClient = PulsarClient
                .Builder()
                .ServiceUrl(new Uri("pulsar://localhost:6650"))
                .RetryInterval(TimeSpan.FromSeconds(3))
                .Build();
            
            // 透過 client 來建立指定處理 message 類型的 Producer
            builder.Services.AddSingleton( _ => 
                pulsarClient.NewProducer(Schema.String)
                .Topic("persistent://public/default/yowkotest")//需指定 topic 名稱，格式：`{persistent|non-persistent}://tenant/namespace/topic`
                .Create());
            ```

        - 不使用 builder 建立

             ```cs
            // 建立 Pulsar 連線 client，可以透過 `ServiceUrl` 設定 Pulsar 位址(預設： `pulsar://localhost:6650`)、`RetryInterval` 設定重試或是重連的間隔秒數 (預設值：3s)
            var pulsarClient = PulsarClient
                .Builder()
                .ServiceUrl(new Uri("pulsar://localhost:6650"))
                .RetryInterval(TimeSpan.FromSeconds(3))
                .Build();
            
            // 透過 client 來建立 Producer
            builder.Services.AddSingleton( _ =>
            {
                //需指定 topic 名稱(格式：`{persistent|non-persistent}://tenant/namespace/topic`) 與 producer 處理的 message 類型
                var options = new ProducerOptions<string>("persistent://public/default/yowkotest",Schema.String);
                return pulsarClient.CreateProducer(options);
            });
            ```

    - 實際使用

        - 取得 Producer instance

            > producer instance 需要與註冊時有一致的 message 類型 (以下以 `string` 為例)

            ```cs
            private readonly IProducer<string> _pulsarProducer;

            public Controller(IProducer<string> pulsarProducer)
            {
                _pulsarProducer = pulsarProducer;
            }
            ```

        - 發送訊息

            ```cs
            var data = "Hello World";
            await _pulsarProducer.Send(new MessageMetadata(),data);
            ```

2. Pulsar.Client

    - Program.cs : 註冊 Pulsar Producer

        ```cs
        const string serviceUrl = "pulsar://localhost:6650";
        const string topicName = "yowkotest2";

        // 建立 Pulsar 連線 client，可以透過 `ServiceUrl` 設定 Pulsar 位址
        var client = await new PulsarClientBuilder()
            .ServiceUrl(serviceUrl)
            .BuildAsync();
         // 透過 client 來建立 Producer
        builder.Services.AddSingleton( _ =>  client
            //指定處理 message 類型的 Producer
            .NewProducer(Schema.STRING(Encoding.UTF8))
            // topic 可以是單純的 topic name
            .Topic(topicName)
            .CreateAsync().Result);
        ```

    - 實際使用

        - 取得 Producer instance

            > producer instance 需要與註冊時有一致的 message 類型 (以下以 `string` 為例)

            ```cs
            private readonly IProducer<string> _pulsarProducer;

            public Controller(IProducer<string> pulsarProducer)
            {
                _pulsarProducer = pulsarProducer;
            }
            ```

        - 發送訊息

            ```cs
            var data = "Hello World";
            await _pulsarProducer.SendAsync(data);
            ```

## 心得

- 關於 DotPulsar

    1. 官網文件與 GitHub 的 wiki 不是即時的

        > library 的 api 已更新，官網上的教學未更新，照著做一定沒辦法用

        - 沒有指定 message 類型

            ![1DotPulsarmissingtmessage](https://user-images.githubusercontent.com/3851540/157363861-e4700d79-ec77-451d-b5ce-edacd164d337.png)

        - 使用錯誤的 message 類型、方法未傳遞 meta

            ![2DotPulsarnometa](https://user-images.githubusercontent.com/3851540/157363866-f5a7ebf6-cfc5-4954-a0bf-84f23a3eacec.png)

    2. GitHub repo 的 readme 與 samples 的程式碼勉強可用 (需要小調整)，但只有 builder 寫法

        ![3DotPulsarnometa](https://user-images.githubusercontent.com/3851540/157363869-68fb7747-6826-491c-bb8e-d15d7c19cdd9.png)

    3. 方法命名不好： async 方法，名稱未以 async 結尾

        ![4DotPulsarmethodname](https://user-images.githubusercontent.com/3851540/157363874-9c2a70c4-1681-4af2-9b81-f7c949122a3b.png)

- 關於 Pulsar.Client

    1. sample code 沒有指定 message 類型

        ![5PulsarClientmissingtmessage](https://user-images.githubusercontent.com/3851540/157833089-db69f814-f7e8-41c9-ba01-e7706ffbc865.png)

    2. 文件相較於 DotPulsar 沒那麼完整(相對出錯機會也小？)
    3. 建立 producer 只有 async 方式，在註冊時可能會造成 di 無法正確取得 service
    4. 使用上較直覺

        - topic 沒有多餘的意外
        - 發送訊息時也不需要額外傳遞 meta

    5. NuGet 上的下載數與 GitHub 上的 star 數都是 Pulsar.Client 多一些

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
9. [在 ASP.NET Core 中從 Apache Pulsar 接收訊息](/aspdotnet-core-pulsar-consumer)
10. [在 ASP.NET Core 中從 Apache Pulsar 接收訊息 (Reader)](/aspdotnet-core-pulsar-reader)
