---
title: "使用 C# 搭配 Avro 存取 Kafka"
date: 2023-07-10T00:30:00+08:00
lastmod: 2023-07-10T00:30:31+08:00
draft: false
tags: ["csharp","kafka"]
slug: "csharp-avro-kafka"
---

## 使用 C# 搭配 Avro 存取 Kafka

之前筆記 [使用 Docker Compose 啟動 Avro Schema Registry](/docker-compose-avro-schema-registry/) 紀錄到如何使用 docker compose 來快速建立 Kafka 與 Schema Registry，當然沒事不會特別建立環境，今天就進入重點，紀錄一下該如何使用 C# 搭配 Avro 存取 Kafka

## 基本環境說明

1. macOS Ventura 13.4.1
2. .NET SDK 6.0.400
3. JetBrains Rider 2023.1.3
4. OrbStack 0.13.0(1910)
5. NuGet Package

    - Confluent.Kafka 2.1.1
    - Confluent.SchemaRegistry 2.1.1
    - Confluent.SchemaRegistry.Serdes.Avro 2.1.1

6. container images
    - quay.io/strimzi/kafka:latest-kafka-3.5.0-amd64
    - confluentinc/cp-schema-registry:7.4.0
    - landoop/schema-registry-ui:0.9.4
7. dotnet tools: Apache.Avro.Tools 1.11.2

    > 用來將 Avro schemas 轉為 C# class

    ```bash
    dotnet tool install --global Apache.Avro.Tools
    ```

8. Kafka 與 Schema Registry 建立

    > 完整說明請參考之前筆記 [使用 Docker Compose 啟動 Avro Schema Registry](/docker-compose-avro-schema-registry/)，其中 `advertised.listeners=PLAINTEXT://192.168.80.3:9092` 記得改為自己的 ip

    - server.properties

        ```yml
        process.roles=broker,controller
        node.id=1
        offsets.topic.replication.factor=1
        controller.quorum.voters=1@kafka:9093
        listeners=PLAINTEXT://:9092,CONTROLLER://:9093
        advertised.listeners=PLAINTEXT://192.168.80.3:9092
        controller.listener.names=CONTROLLER
        listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
        log.dirs=kafka_data
        ```

    - docker-compose.yml

        ```yaml
        version: "3"
    
        services:
          kafka:
            container_name: kafka
            user: root
            image: quay.io/strimzi/kafka:latest-kafka-3.5.0-amd64
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
          schema-registry:
            image: confluentinc/cp-schema-registry:7.4.0
            hostname: schema-registry
            container_name: schema-registry
            depends_on:
              - kafka
            ports:
              - "8081:8081"
            environment:
              SCHEMA_REGISTRY_HOST_NAME: schema-registry
              SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'kafka:9092'
          #schema-registry-ui:
          #  image: landoop/schema-registry-ui
          #  container_name: schema-registry-ui
          #  depends_on:
          #    - schema-registry
          #    - kafka
          #  ports:
          #    - "8000:8000"
          #  environment:
          #    - SCHEMAREGISTRY_URL=http://schema-registry:8081
          #    - PROXY=true
        ```

## 步驟說明

1. 定義 Avro schemas

    - User.avsc

        ```avsc
        {
          "type": "record",
          "name": "User",
          "namespace": "KafkaAvro",
          "fields": [
            {
              "name": "id",
              "type": "int"
            },
            {
              "name": "name",
              "type": "string"
            },
            {
              "name": "age",
              "type": "int"
            }
          ]
        }
        ```

2. 將 Avro schemas 轉換為 C# class

    - 語法

        ```bash
        avrogen -s <schemafile> <outputdir>
        ```

    - 範例

        ```bash
        avrogen -s User.avsc .
        ```

3. Produce

    ```cs
    var producerConfig = new ProducerConfig()
    {
        //透過 local ip 來連線 kafka
        BootstrapServers = "192.168.80.3:9092"
    };

    var schemaRegistryConfig = new SchemaRegistryConfig()
    {
        Url = "http://localhost:8081"
    };
    var avroSerializerConfig = new AvroSerializerConfig
    {
        BufferBytes = 100
    };
    const string topicName = "test-topic";
    CancellationTokenSource cts = new CancellationTokenSource();
    using var schemaRegistry = new CachedSchemaRegistryClient(schemaRegistryConfig);
    using var producer =
        new ProducerBuilder<string, User>(producerConfig)
            .SetValueSerializer(new AvroSerializer<User>(schemaRegistry, avroSerializerConfig))// produce 時會將 Avro schema 註冊至 Schema Registry，schema name 會是 {topicName}-value
            .Build();

    User user = new User
    {
        id = 1,
        name = "Yowko Tsai",
        age = 40
    };
    await producer
        .ProduceAsync(topicName, new Message<string, User> { Key = topicName, Value = user })
        .ContinueWith(task =>
        {
            if (!task.IsFaulted)
            {
                Console.WriteLine($"produced to: {task.Result.TopicPartitionOffset}");

                return;
            }

            Console.WriteLine($"error producing message: {task.Exception?.InnerException}");
        });
    cts.Cancel();
    ```

4. Consume

    ```cs
    var schemaRegistryConfig = new SchemaRegistryConfig()
    {
        Url = "http://localhost:8081"
    };
    var consumerConfig = new ConsumerConfig
    {
        BootstrapServers = "192.168.80.3:9092",
        GroupId = "avro-consumer",
        AutoOffsetReset = AutoOffsetReset.Earliest
    };
    const string topicName = "test-topic";
    CancellationTokenSource cts = new CancellationTokenSource();

    using var schemaRegistry = new CachedSchemaRegistryClient(schemaRegistryConfig);
    using var consumer =
        new ConsumerBuilder<string, User>(consumerConfig)
            .SetValueDeserializer(new AvroDeserializer<User>(schemaRegistry).AsSyncOverAsync())
            .SetErrorHandler((_, e) => Console.WriteLine($"Error: {e.Reason}"))
            .Build();
    consumer.Subscribe(topicName);

    try
    {
        while (true)
        {
            try
            {
                var consumeResult = consumer.Consume(cts.Token);
                var user = consumeResult.Message.Value;
                Console.WriteLine(
                    $"key: {consumeResult.Message.Key}, user id: {user.id}, user name: {user.name}, user age: {user.age}");
            }
            catch (ConsumeException e)
            {
                Console.WriteLine($"Consume error: {e.Error.Reason}");
            }
        }
    }
    catch (OperationCanceledException)
    {
        consumer.Close();
    }
    ```

## 心得

1. 相關分享資源較少，比較少人紀錄整個使用流程，不確定是因為太少人用而沒有網路文章還是剛好用的都不是 C#
2. 個人覺得資料交換多個 Schema Registry 有點麻煩，多個 single point of failure

完整程式碼：[yowko/csharp-avro-kafka](https://github.com/yowko/csharp-avro-kafka)

## 參考資訊

1. [使用 Docker Compose 啟動 Avro Schema Registry](/docker-compose-avro-schema-registry/)
2. [Create C# Classes from Avro IDL](https://dev.to/cainux/create-c-classes-from-avro-idl-1f84)
3. [confluent-kafka-dotnet/examples/AvroSpecific/Program.cs](https://github.com/confluentinc/confluent-kafka-dotnet/blob/master/examples/AvroSpecific/Program.cs)
4. [yowko/csharp-avro-kafka](https://github.com/yowko/csharp-avro-kafka)
