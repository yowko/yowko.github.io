---
title: "RabbitMQ 關於寫入部份 Queue 的權限設定"
date: 2022-03-12T00:30:00+08:00
lastmod: 2022-03-12T00:30:31+08:00
draft: false
tags: ["RabbitMQ","csharp"]
slug: "rabbitmq-topic-permission-queue"
---

## RabbitMQ 關於寫入部份 Queue 的權限設定

之前筆記 [RabbitMQ 為不同帳號設定不同 Queue 權限](/rabbitmq-user-queue-authorisation/) 紀錄到讓不同 user 有不同權限，當時的情境是以讀取資料為例，最近同事想要為寫入也加上不同權限設定，依照之前的做法卻無法成功生效，我重新確認後發現之前筆記觀念不是完全正確，趁這個機會梳理一下規則，希望下次不要再打自己臉了XD

## 基本環境說明

1. macOS Monterey 12.2.1
2. docker desktop 4.2.0(70708)
3. docker images

    - rabbitmq:3.9.13-management

4. .NET SDK 6.0.200
5. NuGet packages

    - RabbitMQ.Client 6.2.1

6. 簡化情境說明

    1. 一個 exchange 在 vhost root 下：yowkoex
    2. 兩個 queue binding 在 yowkoex 中：q1 (routing_key:1)；q2 (routing_key:2)
    3. user：yowko 只有寫 q1 與讀 q2 的權限

7. rabbitmq 環境建立

    - 使用 docker 啟動 rabbitmq

        ```bash
        docker run -d --rm --name rabbitmq -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=pass.123 rabbitmq:3.9.13-management
        ```

    - 建立 user: yowko

        ```bash
        rabbitmqctl add_user yowko pass.123
        ```

    - 建立 exchange: yowkoex

        ```bash
        rabbitmqadmin declare exchange name=yowkoex type=topic -u admin -p pass.123
        ```

    - 建立 queue

        - q1

            ```bash
            rabbitmqadmin declare queue name=q1 durable=true -u admin -p pass.123
            ```

        - q2

            ```bash
            rabbitmqadmin declare queue name=q2 durable=true -u admin -p pass.123
            ```

    - 將 exchange 與 queue 做 binding

        - q1

            ```bash
            rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="q1" routing_key="1" -u admin -p pass.123
            ```

        - q2

            ```bash
            rabbitmqadmin declare binding source="yowkoex" destination_type="queue" destination="q2" routing_key="2" -u admin -p pass.123
            ```

8. 存取訊息程式碼

    ```cs
    static void Main(string[] args)
        {
            PublishMessage();

            //ConsumeMessage();
        }

        private static void PublishMessage()
        {
            var factory = new ConnectionFactory() { Uri = new Uri("amqp://yowko:pass.123@localhost:5672/") };
            using (var connection = factory.CreateConnection())
            using (var channel = connection.CreateModel())
            {
                var message = $@"Hello World! @{DateTime.Now}";
                var body = Encoding.UTF8.GetBytes(message);
                var props = channel.CreateBasicProperties();

                channel.BasicPublish(exchange: "yowkoex",
                    routingKey: "1",
                    basicProperties: props,
                    body: body);

                Console.WriteLine($"[x] Sent {message}");
            }

            Console.WriteLine("Press [enter] to exit.");
            Console.ReadLine();
        }
        
        private static void ConsumeMessage()
        {
            var factory = new ConnectionFactory() { Uri = new Uri("amqp://yowko:pass.123@localhost:5672/") };

            using var connection = factory.CreateConnection();
            using var channel = connection.CreateModel();
            
            var consumer = new EventingBasicConsumer(channel);
            consumer.Received += (model, ea) =>
            {
                var body = ea.Body.ToArray();
                var message = Encoding.UTF8.GetString(body);
                Console.WriteLine(" [x] Received {0}", message);
            };
            
            channel.BasicConsume(queue: "q2",
                autoAck: true,
                consumer: consumer);

            Console.WriteLine(" Press [enter] to exit.");
            Console.ReadLine();
        }
    ```

## 設定與說明

1. 設定 user 權限

    ![1setpermission](https://user-images.githubusercontent.com/3851540/158097854-1e6c8d5a-5f80-49a4-bc76-89ab1245917e.png)

    ```bash
    rabbitmqctl set_permissions -p / yowko "" "yowkoex" "q2"
    ```

    > 針對存取的 resource 來做設定，寫入是對 `exchange`，讀取是對 `queue`

2. 設定 user topic 權限

    ![2settopicpwermission](https://user-images.githubusercontent.com/3851540/158097861-85d576f8-b700-4e97-a453-183aca5b6a58.png)

    ```bash
    rabbitmqctl set_topic_permissions -p / yowko yowkoex "1" ""
    ```

    > 針對指定 exchange 加上寫入的 routing key 限制，不過我不理解的為什麼讀取都是直接對 queue 怎麼會出現 rouiting key 限制設定

## 心得

1. 嘗試發送訊息到沒有寫入權限的 q2 (routing_key 帶 `2`) 不會拋出錯誤
2. 嘗試從沒有讀取權限的 q1 接收訊息會出現錯誤

    - 錯誤訊息

        ```txt
        Unhandled exception. RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Peer, code=403, text='ACCESS_REFUSED - access to queue 'q1' in vhost '/' refused for user 'yowko'', classId=60, methodId=20
        ```

    - 錯誤截圖

        ![3error](https://user-images.githubusercontent.com/3851540/158097865-c72b46ac-26b2-4754-a757-ce8eae030a8b.png)

## 參考資訊

1. [RabbitMQ 為不同帳號設定不同 Queue 權限](/rabbitmq-user-queue-authorisation/)
