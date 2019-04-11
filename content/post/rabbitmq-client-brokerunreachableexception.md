---
title: "使用 RabbitMQ.Client 連線至 RabbitMQ 出現 BrokerUnreachableException"
date: 2017-05-18T23:30:00+08:00
lastmod: 2018-09-22T23:30:09+08:00
draft: false
tags: ["Debug","RabbitMQ"]
slug: "rabbitmq-client-brokerunreachableexception"
aliases:
    - /2017/05/rabbitmq-client-brokerunreachableexception.html
---
# 使用 RabbitMQ.Client 連線至 RabbitMQ 出現 BrokerUnreachableException
因為專案需要就開始動手來嘗試各種 MQ 的比較，第一個開始測試的就是 RabbitMQ，原本打算先做幾個基本測試：

1.  試打訊息進 queue
2.  讓 RabbitMQ 分派訊息給其他 client
3.  client 正確接受訊息


完成基本測試後就可以紀錄一下過程中使用的工具與方法，想不到第一步就卡住XD 連線一直出現問題，所以第一篇 RabbitMQ 的紀錄文反而是 debug

## 問題發生原因

RabbitMQ 使用 virtual host 來管理用戶權限，virtual host 可以視為一種 namespace 的概念，預設的 virtual host 是 `/` ,新增 user 預設不開放存取權限。


確認方式

1.  登入管理後台：`http://localhost:15672/`
2.  Admin --> Users --> 檢查 user 是否可以 access 指定的 virtual host

    ![1useraccess](https://cloud.githubusercontent.com/assets/3851540/26199999/d4fdc9fe-3bfe-11e7-97a4-8d7bb4390bc0.png)

## 錯誤訊息

- 訊息內容

    ``` 
    BrokerUnreachableException: None of the specified endpoints were reachable
    ```

- 錯誤截圖

    ![2errormsg](https://cloud.githubusercontent.com/assets/3851540/26198893/6fb57622-3bfa-11e7-9cfd-7eeeac68767b.png)

## 解決方式

設定 user 有存取指定 virtual hosts 的權限

*   開啟 `RabbitMQ Command Prompt (sbin dir)` (安裝 RabbitMQ 時預設裝進來的工具)
*   設定權限語法

    ```
    rabbitmqctl set_permissions -p {virtual host} {userName} "." "." ".*"
    ```

*   範例：開放 `/` 的權限給 `yowko`

    ```
    rabbitmqctl set_permissions -p / yowko "." "." ".*"
    ```

    ![3geant](https://cloud.githubusercontent.com/assets/3851540/26198890/6f6e2de4-3bfa-11e7-9855-45ef03786b66.png)

*   設定完成

    ![4done](https://cloud.githubusercontent.com/assets/3851540/26198891/6f935aa6-3bfa-11e7-8a8c-c603df72fd4f.png)

# 參考資訊

1.  [RabbitMQ原理與相關操作(一)](http://www.cnblogs.com/ericli-ericli/p/5917018.html)
