---
title: "如何在 Windwos 上設定 RabbitMQ Cluster"
date: 2017-08-09T23:57:00+08:00
lastmod: 2020-12-11T23:57:06+08:00
draft: false
tags: ["RabbitMQ"]
slug: "rabbitmq-cluster-on-windows"
aliases:
    - /2017/08/windwos-rabbitmq-cluster.html
---
# 如何在 Windwos 上設定 RabbitMQ Cluster
現在的系統都需要考慮大流量的情境，而系統中承載力最差的節點也就是系統的整體承載量。一般的概念就是哪個節點扛不住就加那個節點的 server (除了 db 之外)，而 message queue 本身是所有訊息都會經過的一個 component，很容易是效能瓶頸點，所以如果預期使用量很高時或是想要達成 HA - High availability 就需要為 RabbitMQ 加入 Cluster 機制

最近剛好為專案設定 RabbitMQ 的 Cluster，需要注意的地方還不少，一定要紀錄一下不然相信很快就會忘記了

## 環境前提

請先安裝好兩組 RabbitMQ，如何在 Windows 安裝 RabbitMQ 請參考 [在 Windows 7、Winodws 10、Windows Server 2016 上安裝 RabbitMQ](/2017/05/install-rabbitmq-on-windows7-windows10-windows2016.html)

## 設定 RabbitMQ Cluster

> 以下動作在 RabbitMQ 的 slave 上操作

1.  stop service

    > 如未成功停止 service 會造成後續動作認證失敗

    *   指令

        > `sc stop RabbitMQ`

    *   GUI

        ![3stopservice](https://user-images.githubusercontent.com/3851540/29130723-4b9417b8-7d5d-11e7-8bf7-8b1489830185.png)

2.  改 cookie

    > 為了讓各組 RabbitMQ 可以認識彼此，請選定其中一組 RabbitMQ 將其 cookie 複製至其他 RabbitMQ 中使用

    *   位置


        *   `C:\Windows\.erlang.cookie`

            > 供 RabbitMQ service 使用

        *   `C:\Users\Current User\.erlang.cookie (%HOMEDRIVE% + %HOMEPATH%\.erlang.cookie)`or `C:\Documents and Settings\Current User\.erlang.cookie`

    *   上述兩個 cookie 內容需一致

3.  restart service

    *   指令

        > `sc start RabbitMQ`

    *   GUI

        ![4restartservice](https://user-images.githubusercontent.com/3851540/29130724-4b986372-7d5d-11e7-9d0a-827d4f8d2f5f.png)

    *   如果出現錯誤訊息，表示修改 cookie 前未成功停止 service，請恢復舊 cookie 再重新操作一次

        ```
            C:\Program Files\RabbitMQ Server\rabbitmq_server-3.6.9\sbin>rabbitmqctl.bat stop_app
            Stopping rabbit application on node 'rabbit@DESKTOP-1JHUOUJ' ...
            Error: unable to connect to node 'rabbit@DESKTOP-1JHUOUJ': nodedown

            DIAGNOSTICS
            ===========

            attempted to contact: ['rabbit@DESKTOP-1JHUOUJ']

            rabbit@DESKTOP-1JHUOUJ:
            * connected to epmd (port 4369) on DESKTOP-1JHUOUJ
            * epmd reports node 'rabbit' running on port 25672
            * TCP connection succeeded but Erlang distribution failed

            * Authentication failed (rejected by the remote node), please check the Erlang cookie


            current node details:
            - node name: 'rabbitmq-cli-86@DESKTOP-1JHUOUJ'
            - home dir: C:\Users\yowko
            - cookie hash: OCvdsgdfggdsfgsdfgYxA==
        ```

    *   錯誤畫面

        ![1authdfailed](https://user-images.githubusercontent.com/3851540/29130721-4b91b2f2-7d5d-11e7-93f3-3ab59505a0f6.png)

    *   重啟後成功

        ![2authpass](https://user-images.githubusercontent.com/3851540/29130722-4b91e36c-7d5d-11e7-8577-b653ec738d44.png)

4.  停止 RabbitMQ

    *   開啟 RabbitMQ 附屬的 Command 工具 - `RabbitMQ Command Prompt (sbin dir)`

        ![5rabbitmqcommand](https://user-images.githubusercontent.com/3851540/29130726-4badf43a-7d5d-11e7-8455-5661d79ae5c2.png)

    *   執行停止 rabbitmq

        > `rabbitmqctl stop_app`

5.  加入 Cluster

    *   繼續使用 RabbitMQ 附屬的 Command 工具 - `RabbitMQ Command Prompt (sbin dir)`
    *   將該 rabbitmq 加至 master 的 cluster 中

        *   語法

            > `rabbitmqctl join_cluster rabbit@{MACHINENAME}`

        *   範例

            > `rabbitmqctl join_cluster rabbit@YowkoPC`

6.  重啟 RabbitMQ

    *   繼續使用 RabbitMQ 附屬的 Command 工具 - `RabbitMQ Command Prompt (sbin dir)`
    *   執行重啟 rabbitmq

        > `rabbitmqctl start_app`

7.  完成設定

> 管理介面上會有多個 node

![6nodes](https://user-images.githubusercontent.com/3851540/29130725-4b9ae552-7d5d-11e7-8648-37e627884910.png)

## 心得

RabbitMQ 的 Windows Cluster 設定在官網上寫得並不是很直覺，找了不少資料也測試好幾次才成功，但好景不常架起來沒多少，測試幾次後又 整個 crash，後來重灌了好幾次才真正搞定

這樣一來不得不擔心，如果有天在 production 出問題會不會解不了 XD

# 參考資訊

1.  [Highly Available (Mirrored) Queues](https://www.rabbitmq.com/ha.html)
2.  [RabbitMQ概念及環境搭建（四）RabbitMQ High Availability](http://blog.csdn.net/zyz511919766/article/details/41896823)
3.  [Setting up RabbitMQ clustering on Windows](http://tammadge.net/2016/07/setting-up-rabbitmq-clustering-on-windows/)
4.  [RabbitMQ – Customize Manual Installation for Windows](https://www.codeproject.com/Articles/1163242/RabbitMQ-Customize-Manual-Installation-for-Windows)
