---
title: "使用 .Net client 連線至 RabbitMQ 收發訊息"
date: 2017-05-22T23:35:00+08:00
lastmod: 2018-09-22T23:35:08+08:00
draft: false
tags: ["RabbitMQ","C#","套件"]
slug: "rabbitmq-client-send-consume"
aliases:
    - /2017/05/rabbitmq-client-send-consume.html
    - /2017/may/rabbitmq-client-send-consume/
---
# 使用 .Net client 連線至 RabbitMQ 收發訊息
繼之前 [使用 RabbitMQ.Client 連線至 RabbitMQ 出現 BrokerUnreachableException](//blog.yowko.com/2017/05/rabbitmq-client-brokerunreachableexception.html) 問題解決後，終於又可以繼續測試 RabbitMQ 了，要比較的不僅僅是 mq 的能力，也要比較使用上的便利性及周邊管理功能或是其他套件的完整度，今天就先紀錄一下 RabbitMQ 的基本操作：收發訊息

## 安裝 RabbitMQ.Client

RabbitMQ 官方的介紹 [.NET/C# RabbitMQ client library](https://www.rabbitmq.com/dotnet.html)，說明很完整

![1PLUGIN](https://cloud.githubusercontent.com/assets/3851540/26316464/ee998682-3f46-11e7-8b45-1f9b18d88891.png)

## 接收訊息

> 如果有多個 consumer (接受端)，記得要先啟動 consumer，否則訊息將都會由第一啟動的 consumer 獨佔

> 如果啟動接收訊息時出現錯誤，請先參考 [使用 RabbitMQ.Client 連線至 RabbitMQ 出現 BrokerUnreachableException](//blog.yowko.com/2017/05/rabbitmq-client-brokerunreachableexception.html)

- 程式碼

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

## 發送訊息

> 如果啟動發送訊息時出現錯誤，請先參考 [使用 RabbitMQ.Client 連線至 RabbitMQ 出現 BrokerUnreachableException](//blog.yowko.com/2017/05/rabbitmq-client-brokerunreachableexception.html)

- 程式碼

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

## 心得

使用上還算便利，但很多參數還不知道如何使用，有較深入研究使用會再另外紀錄介紹，敬請期待

# 參考資訊

1.  [使用 RabbitMQ.Client 連線至 RabbitMQ 出現 BrokerUnreachableException](//blog.yowko.com/2017/05/rabbitmq-client-brokerunreachableexception.html)
2.  [.NET/C# RabbitMQ client library](https://www.rabbitmq.com/dotnet.html)
3.  [RabbitMQ原理與相關操作(一)](http://www.cnblogs.com/ericli-ericli/p/5917018.html)
