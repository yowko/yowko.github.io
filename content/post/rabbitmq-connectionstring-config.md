---
title: "將 RabbitMQ 的連線參數移至 Config 中"
date: 2017-07-26T23:26:00+08:00
lastmod: 2021-10-28T23:26:50+08:00
draft: false
tags: ["RabbitMQ","web.config"]
slug: "rabbitmq-connectionstring-config"
aliases:
    - /2017/07/rabbitmq-connectionstring-config.html
---
## 將 RabbitMQ 的連線參數移至 Config 中

之前文章 [使用 .Net client 連線至 RabbitMQ 收發訊息](/2017/05/rabbitmq-client-send-consume.html) 介紹了如何連線至 RabbitMQ 收發訊息，不過當時直接在程式碼指定了相關的連線資訊，身為一個優秀的工程師都曉得連線字串這東西絕對不能寫死在程式碼中，不然如果有一天需要修改時，會改到哭出來，所以我們應該將連線字串移至 config 中來共用，另外也會將 deueue 用法改使用新的 api (舊 api 已被標記為 Deprectated)，除此之外也將原本散落各處的名稱定義(e.g. queue,exchange,routingkey...)做個整理

今天修改的部份並不多，但連線字串的寫法有點特別，一定要紀錄下來避免忘記，加上日後需要才能快速找到東西 c.c.

## 原程式碼

1. Consumer

    ```cs
    void Main()
    {
        //初始化連線資訊
        var factory = new ConnectionFactory()
        {
            //設定 RabbitMQ 位置
            HostName = "localhost",
            //設定 RabbitMQ port
            Port = 5672,
            //設定連線 RabbitMQ username
            UserName = "yowko",
            //設定 RabbitMQ password
            Password = "pass.123"
        };
        //開啟連線
        using (var connection = factory.CreateConnection())
        //開啟 channel
        using (var channel = connection.CreateModel())
        {
            //宣告 queues
            channel.QueueDeclare("yowkoTest", false, false, false, null);
            //建立 consumer
            var consumer = new QueueingBasicConsumer(channel);
            channel.BasicConsume("yowkoTest", true, consumer);
            Console.WriteLine(" waiting for message.");
            //持續等著接收訊息
            while (true)
            {
                //從 RabbitMQ 取得訊息
                var ea = (BasicDeliverEventArgs)consumer.Queue.Dequeue();
                            var body = ea.Body;
                var message = Encoding.UTF8.GetString(body);
                Console.WriteLine("Received {0}", message);
                Thread.Sleep(5000);
            }
        }
    }
    ```

2. Publisher

    ```cs
    void Main()
    {
        //初始化連線資訊
        var factory = new ConnectionFactory();
        //設定 RabbitMQ 位置
        factory.HostName = "localhost";
        //設定 RabbitMQ port
        factory.Port = 5672;
        //設定連線 RabbitMQ username
        factory.UserName = "yowko";
        //設定 RabbitMQ password
        factory.Password = "pass.123";
                    
        //開啟連線
        using (var connection = factory.CreateConnection())
        //開啟 channel
        using (var channel = connection.CreateModel())
        {
            //宣告 exchanges，RabbitMQ提供了四種Exchange模式：fanout,direct,topic,header
            channel.ExchangeDeclare("yowko", ExchangeType.Fanout);
            //宣告 queues
            channel.QueueDeclare("yowkoTest", false, false, false, null);
            //將 exchnage、queue 依 route rule 綁定
            channel.QueueBind("yowkoTest", "yowko", "hello", null);
            //  channel.QueueDeclare("hello", false, false, false, null);
            string message = $"Hello World-{Guid.NewGuid()}";
            var body = Encoding.UTF8.GetBytes(message);
            channel.BasicPublish("yowko", "hello", null, body);
            Console.WriteLine(" set {0}", message);
        }
    }
    ```

## 將連線字串移至 config

* Pattern

    > `amqp://{帳號}:{帳碼}@{rabbitMQ_url}:{rabbitMQ_port}`

* 實際寫法

    > `amqp://yowko:pass.123@localhost:5672`

## 使用 EventHandler 來調整 Consumer

```cs
var consumer = new EventingBasicConsumer(channel);
consumer.Received += (model, ea) =>
{
    var body = ea.Body;
    var message = Encoding.UTF8.GetString(body);
    Console.WriteLine($"Received： {message}");
    channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
    Console.WriteLine("OK");
};
```

## 修改後程式碼

1. Consumer

    ```cs
    void Main()
    {
        //初始化連線資訊
        var factory = new ConnectionFactory();
        factory.Uri = "amqp://yowko:pass.123@localhost:5672";
        var queueName = "yowkoTest";
        
        using (var connection = factory.CreateConnection())
        using (var channel = connection.CreateModel())
        {
            channel.QueueDeclare(queue: queueName,
                durable: true,
                exclusive: false,
                autoDelete: false,
                arguments: null);
            channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
            Console.WriteLine(" [*] Waiting for messages.");
            var consumer = new EventingBasicConsumer(channel);
            consumer.Received += (model, ea) =>
            {
                var body = ea.Body;
                var message = Encoding.UTF8.GetString(body);
                Console.WriteLine($"Received： {message}");
                channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
                Console.WriteLine("OK");
            };
            channel.BasicConsume(queue: queueName,
                noAck: false,
                consumer: consumer);
            Console.WriteLine(" Press [enter] to exit.");
            Console.ReadLine();
        }
    }
    ```

2. Publisher

    ```cs
    void Main()
    {
        //初始化連線資訊
        var factory = new ConnectionFactory();
        factory.Uri = "amqp://yowko:pass.123@localhost:5672";
        //開啟連線
        using (var connection = factory.CreateConnection())
        //開啟 channel
        using (var channel = connection.CreateModel())
        {
            string exchange = "yowko";
            string queue = "yowkoTest";
            string routingKey = "hello";
            //宣告 exchanges，RabbitMQ提供了四種Exchange模式：fanout,direct,topic,header
            channel.ExchangeDeclare(exchange, ExchangeType.Fanout);
            //宣告 queues
            channel.QueueDeclare(queue, true, false, false, null);
            //將 exchnage、queue 依 route rule 綁定
            channel.QueueBind(queue, exchange, routingKey, null);
            channel.BasicQos(0, 1, true);
            string message = $"Hello World-{Guid.NewGuid()}";
            var body = Encoding.UTF8.GetBytes(message);
            channel.BasicPublish(exchange, routingKey, new RabbitMQ.Client.Framing.BasicProperties { Persistent = true }, body);
            Console.WriteLine($"Send Message：{message}");
        }
    }
    ```

## 心得

原本只是用來進行 POC 的程式碼，因為要實際應用在 production 環境上，所以需要考量到良好的可閱讀性及較佳的可維護性，在 code review 後發展出較佳的寫法，這讓我想到或許不一定每次都能有其他團隊成員協助 code review ，但仍然可以自行 code review 以確認是否滿足團隊的規範，持續自我要求，漸漸地就會養成寫出較佳程式碼的習慣

## 參考資訊

1. [使用 .Net client 連線至 RabbitMQ 收發訊息](/2017/05/rabbitmq-client-send-consume.html)
2. [Connecting to RabbitMQ](https://help.compose.com/v2.0/docs/rabbitmq-connecting-to-rabbitmq)
