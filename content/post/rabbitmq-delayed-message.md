---
title: "讓 RabbitMQ 支援延遲發送訊息"
date: 2017-07-08T18:47:00+08:00
lastmod: 2018-09-23T18:47:18+08:00
draft: false
tags: ["套件","RabbitMQ"]
slug: "rabbitmq-delayed-message"
aliases:
    - /2017/07/rabbitmq-delayed-message.html
---
# 讓 RabbitMQ 支援延遲發送訊息
一般情況下，Message Queue 都是將 message 由 producer 送給 broker 後接著就由 consumer dequeue 進行處理，常見的額外需求是 message 有不同的 priority，但這次客戶提出想要在發生動作的當下不立即觸發對應的行為，想透過設定延遲時間來執行對應行為

客戶的需求永遠是刺激工程師進步的原動力，辜且不論客戶延遲執行的理由跟合理性，身為工程師一定要有可以滿足客戶需求的能力，所以就先想想可能的解決方案

如果是一般情況下可能我會將 message 與預計執行動作的時間紀錄到 db，然後再使用 scheduler 定期去檢查時間及實際執行動作，而這次專案因為已經使用 RabbitMQ 於是就試試 RabbitMQ 是否可以支援這樣的需求，來看看可以怎麼設定吧

## RabbitMQ 安裝套件 - RabbitMQ Delayed Message Plugin

預設情境下 RabbitMQ 並不支援延遲發送的功能，但 RabbitMQ 不愧是套成熟的 mq，可以透過安裝套件來讓 RabbitMQ 支援延遲發送

1.  至 [RabbitMQ Community Plugins page](http://www.rabbitmq.com/community-plugins.html) 下載 [rabbitmq_delayed_message_exchange](https://bintray.com/rabbitmq/community-plugins/rabbitmq_delayed_message_exchange/_latestVersion#files) 套件 (.ez 檔)
2.  將下載的 `.ez` 檔複製至 RabbitMQ 安裝目錄下的

    > 我的檔名是 `rabbitmq_delayed_message_exchange-0.0.1.ez`，我的安裝目錄是 `C:\Program Files\RabbitMQ Server\rabbitmq_server-3.6.9\plugins`

3.  啟用套件 - RabbitMQ Delayed Message Plugin
    *   開啟 RabbitMQ Command Prompt (sbin dir)

        ![1command](https://user-images.githubusercontent.com/3851540/27984600-a3d2fa8a-640c-11e7-9952-907e49c68147.png)

    *   執行 `rabbitmq-plugins enable rabbitmq_delayed_message_exchange` 以啟用套件

        ![2enable](https://user-images.githubusercontent.com/3851540/27984601-a3f7cd38-640c-11e7-9ec0-3980d5651580.png)

## 修改發送程式碼

要延遲發送 message 只需要為 message 加上一個 `x-delay` 的 header 及延遲毫秒數並在發送時指定 property 給 message 即可

*   程式碼說明

    ```cs
    // 建立 BasicProperties
    var props = new BasicProperties();
    // 建立 diciionary 來放 header 內容
    Dictionary<string, object> headers = new Dictionary<string, object>();
    // 指定 x-delay 及 延遲毫秒數
    headers.Add("x-delay", 5000);
    // 將定義完成的 dictionary 指定給 BasicProperties 的 header
    props.Headers = headers;
    // 以 BasicProperties 發送訊息
    channel.BasicPublish(exchange, routingKey, props, body);
    ```

*   完整程式碼

    ```cs
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
        string exchange = "yowkoDelay";
        string queue = "yowkoDelayTest";
        string routingKey = "hello";
        Dictionary<string, object> args = new Dictionary<string, object>();
        args.Add("x-delayed-type", "direct");
        //宣告 exchanges，RabbitMQ提供了四種Exchange模式：fanout,direct,topic,header
        channel.ExchangeDeclare(exchange, "x-delayed-message", true, false, args);
        //宣告 queues
        channel.QueueDeclare(queue, true, false, false, null);
        //將 exchnage、queue 依 route rule 綁定
        channel.QueueBind(queue, exchange, routingKey, null);
        string message = $"delayed payload-{Guid.NewGuid()}";
        var body = Encoding.UTF8.GetBytes(message);
        // 建立 BasicProperties
        var props = new BasicProperties();
        // 建立 diciionary 來放 header 內容
        Dictionary<string, object> headers = new Dictionary<string, object>();
        // 指定 x-delay 及 延遲毫秒數
        headers.Add("x-delay", 5000);
        // 將定義完成的 dictionary 指定給 BasicProperties 的 header
        props.Headers = headers;
        // 以 BasicProperties 發送訊息
        channel.BasicPublish(exchange, routingKey, props, body);
        Console.WriteLine(" set {0}", message);
    }
    ```

*  實際效果

    > 將同一則訊息發送至兩個不同的 queue，一個是直接發送，另一個則使用延遲發送，確認延遲發送可以運作

    ![3result](https://user-images.githubusercontent.com/3851540/27984602-a40d667a-640c-11e7-915c-186d3128e3e5.png)

## 心得

透過安裝套件及修改發送端程式就可以達成延遲發送目的，感覺上還滿方便的，只是我在進行長時間測試時發現，超過一個小時的延遲無法被達成，我自己實際測試的上限都約在一個小時以下，超過一個小時的延遲沒有成功過，但我沒查到實際問題發生原因，官方文件上寫的限制數字是 `ERL_MAX_T` = 4294967295 ，遠大於我測試出來的數字，我看 issues 列表也沒有相關資訊，不知道是不是 windows 版限制，提供給大家參考

# 參考資訊

1.  [rabbitmq/rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)
2.  [Scheduling Messages with RabbitMQ](https://www.rabbitmq.com/blog/2015/04/16/scheduling-messages-with-rabbitmq/)
