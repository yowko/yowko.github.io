---
title: "C# 連線至 RabbitMQ Cluster - 使用 RabbitMQ .Net Client 及 EasyNetQ"
date: 2017-08-20T01:26:00+08:00
lastmod: 2021-11-03T13:50:55+08:00
draft: false
tags: ["套件","RabbitMQ","csharp"]
slug: "dotnet-client-rabbitmq-cluster"
aliases:
    - /2017/08/dotnet-client-rabbitmq-cluster.html
---
## C# 連線至 RabbitMQ Cluster - 使用 RabbitMQ .Net Client 及 EasyNetQ

之前文章 [如何在 Windwos 上設定 RabbitMQ Cluster](/windwos-rabbitmq-cluster) 介紹到透過建立 cluster 的方式來讓 RabbtMQ 可以擁有 HA ，而在 queue 及 message 完整性方面則利用 mirrored queue 的機制來處理，詳細內容可以參考 [設定 RabbitMQ 的 Mirrored Queues - 讓 Queue 內容可以在多組 RabbitMQ 同步](/rabbitmq-mirrored-queues)

基礎建設已經有了雛型，接著就來看看 C# 該如何連線 RabbitMQ cluster 吧

## 使用 RabbitMQ .Net Client

詳細說明請參考 [使用 .Net client 連線至 RabbitMQ 收發訊息](/rabbitmq-client-send-consume)，以下就節錄其中內容重點說明連線 cluster 的差異

1. producer

    ```cs
    //初始化連線資訊
    var factory = new ConnectionFactory();
    //設定 RabbitMQ 位置
    //factory.HostName = "localhost";
    //設定連線 RabbitMQ username
    factory.UserName = "yowko";
    //設定 RabbitMQ password
    factory.Password = "pass.123";
    //寫法二 
    //factory.Uri = "amqp://yowko:pass.123@localhost:5672";
    //寫法三
    //var factory = new ConnectionFactory() { Uri = "amqp://yowko:pass.123@192.168.56.101:5672"};
    //開啟連線
    using (var connection = factory.CreateConnection(new string[2] { "192.168.56.101","localhost"}))
    //開啟 channel
    using (var channel = connection.CreateModel())
    {
        string exchange = "yowko";
        string queue = "eventTest";
        string routingKey = "hello";
        //宣告 exchanges，RabbitMQ提供了四種Exchange模式：fanout,direct,topic,header
        channel.ExchangeDeclare(exchange, ExchangeType.Direct);
        //宣告 queues
        channel.QueueDeclare(queue, true, false, false, null);
        //將 exchnage、queue 依 route rule 綁定
        channel.QueueBind(queue, exchange, routingKey, null);
        channel.BasicQos(0, 1, true);
        string message = $"Hello World-{Guid.NewGuid()}";
        var body = Encoding.UTF8.GetBytes(message);
        channel.BasicPublish(exchange, routingKey, new RabbitMQ.Client.Framing.BasicProperties { Persistent = true }, body);
        Console.WriteLine($"Send Message：{message};{connection.ToString()}");
    }
    ```

    > 變化不大，就是原本在 `ConnectionFactory` 指定連線字串，現在改由 `CreateConnection` 時傳入多個 host 資訊，該連線哪個 host，client 會自行決定

2. consumer

    ```cs
    //初始化連線資訊
    var factory = new ConnectionFactory()
    {
        //設定連線 RabbitMQ username
        UserName = "yowko",
        //設定 RabbitMQ password
        Password = "pass.123",
        //自動回復連線
        AutomaticRecoveryEnabled = true,
        //心跳檢測頻率
        RequestedHeartbeat = 10,
    };
    var queueName = "event";
    //連線多個 rabbitmq instance
    using (var connection = factory.CreateConnection(AmqpTcpEndpoint.ParseMultiple("localhost:5672,192.168.56.101:5672")))
    {
        //處理連線中斷
        connection.ConnectionShutdown += (o, e) =>
        {
            //handle disconnect      
            Console.WriteLine($"Fail:{0},{e}");
        };
        //開啟 channel
        using (var channel = connection.CreateModel())
        {
            //宣告 queues
            channel.QueueDeclare(queue: queueName,
                durable: true,
                exclusive: false,
                autoDelete: false,
                arguments: null);
            
            channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
            Console.WriteLine(" [*] Waiting for messages.");
            //建立 consumer
            var consumer = new EventingBasicConsumer(channel);
            channel.BasicConsume(queue: queueName,
                    noAck: false,
                    consumer: consumer);
            //收到訊息時的處理方式
            consumer.Received += (model, ea) =>
            {
                var body = ea.Body;
                var message = Encoding.UTF8.GetString(body);
                Console.WriteLine($" [x] Received {message} from {connection.ToString()}");
                //手動 ack
                channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
                Console.WriteLine("OK");
            };
            Console.WriteLine(" Press [enter] to exit.");
            //持續等著接收訊息
            while (true)
            {
            }
        }
    }
    ```

    > 連線 RabbitMQ cluster 的方式與 publisher 相同，都是在 `CreateConnection` 時傳入多個 host 資訊(寫法有兩種)，需要特別注意的是 consumer 如果是常駐型連線，需要自行處理 RabbitMQ failover 問題，可以透過 `connection.ConnectionShutdown` 訂閱連線中斷的事件，但之後的重新連線就得自行處理

## 使用 EasyNetQ

`EasyNetQ` 在 GitHub 的 star 數比 `RabbitMQ .Net Client` 高出一倍，想必有一定的水準，相關內容請直接參考 [EasyNetQ/EasyNetQ](https://github.com/EasyNetQ/EasyNetQ)

1. producer

    ```cs
    //建立連線，並透過 `,` 指定多個 host，prefetchcount 指定一次只處理一筆 message
    using (var advancedBus = RabbitHutch.CreateBus("host=localhost,192.168.56.101;username=yowko;password=pass.123;prefetchcount=1").Advanced)
    {
        //定義 exchange
        var exchange = advancedBus.ExchangeDeclare("eventExchange", ExchangeType.Direct);
        //定義 queue
        var queue = advancedBus.QueueDeclare("event");
        //定義 routingkey
        string routingKey = "test";
        //使用 routingkey 綁定 exchange 及 queue
        var binding = advancedBus.Bind(exchange, queue, routingKey, null);
                    
        string message = $"Hello World-{Guid.NewGuid()}";
        var body = Encoding.UTF8.GetBytes(message);
        // publish 訊息，DeliveryMode 是用來設定 message persist (1:non-persistent;2:persistent)
        advancedBus.Publish(exchange, routingKey, false, new MessageProperties { DeliveryMode = 2 }, body);
        Console.WriteLine($"Send Message：{message},{DateTime.Now}");
    }
    ```

2. consumer

    ```cs
    //建立連線，並透過 `,` 指定多個 host，prefetchcount 指定一次只處理一筆 message
    using (var advancedBus = RabbitHutch.CreateBus("host=127.0.0.1,192.168.56.101;username=yowko;password=pass.123;prefetchcount=1").Advanced)
    {
        //定義 exchange
        var exchange = advancedBus.ExchangeDeclare("eventExchange", ExchangeType.Direct);
        //定義 queue
        var queue = advancedBus.QueueDeclare("event");
        //定義 routingkey
        string routingKey = "test";
        //使用 routingkey 綁定 exchange 及 queue
        var binding = advancedBus.Bind(exchange, queue, routingKey, null);
        //使用多執行緒
        //  advancedBus.Consume(queue, (body, properties, info) => Task.Factory.StartNew(() =>
        //  {
        //   var message = Encoding.UTF8.GetString(body);
        //   Console.WriteLine($"Got message: '{message}',{DateTime.Now}");
        //  }));
        //使用單執行緒
        advancedBus.Consume(queue, (body, properties, info) => 
        {
            Console.WriteLine($"dequeue start at:{DateTime.Now}");
            var message = Encoding.UTF8.GetString(body);
            Console.WriteLine($"Got message: '{message}',{DateTime.Now}");
        });
        while (true)
        {
        }
    }
    ```

    > consumer 用法跟 pbulisher 一致，但不用自行處理連線中斷重連，也不必手動回傳 ack，使用上簡潔不少，非常方便

## 心得

經過一番測試，分別使用 `RabbitMQ .Net Client` 與 `EasyNetQ` 對 RabbitMQ 收發訊息，並執行 RabbitMQ failover 來檢視兩個 library 的行為及效果，最後是 `EasyNetQ` 以較簡易的 API 使用雀屏中選，主要是 RabbitMQ failover 時不需自行處理，再來連線字串寫法比較彈性，使用 `EasyNetQ` 方便性絕對比 `RabbitMQ .Net Client` 好上許多

## 參考資訊

1. [如何在 Windwos 上設定 RabbitMQ Cluster](/windwos-rabbitmq-cluster)
2. [設定 RabbitMQ 的 Mirrored Queues - 讓 Queue 內容可以在多組 RabbitMQ 同步](/rabbitmq-mirrored-queues)
3. [EasyNetQ/EasyNetQ](https://github.com/EasyNetQ/EasyNetQ)
