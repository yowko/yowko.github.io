---
title: "RabbtiMQ Cluster failover 後無法啟動 RabbitMQ instance"
date: 2017-08-20T17:37:00+08:00
lastmod: 2018-09-25T17:37:16+08:00
draft: false
tags: ["Debug","RabbitMQ"]
slug: "rabbtimq-cluster-force-boot"
aliases:
    - /2017/08/rabbtimq-cluster-force-boot.html
---
# RabbtiMQ Cluster failover 後無法啟動 RabbitMQ instance
之前測試 RabbtiMQ Cluster 設定及 EasyNetQ 連線時，反覆進行幾次 node shutdown， RabbitMQ 都可以正常提供服務，EasyNetQ 只有在 failover 後的第一次收發訊息會比平常多花個幾秒鐘處理外，其他也沒有異狀，直到有次測試將其中一個 node shutdown 後就下班，隔天發現時卻無法重新啟動 XD

經過反覆測試後，發現如果是長時間失去連線，會造成該 node 無法重新加入 cluster，至於原因還不明，可能得待我的 linux 環境架好再來確認是否因為 windows 環境所造成的，目前就先紀錄一下解決方式

## 錯誤訊息

*   訊息內容

    ```
        C:\Program Files\RabbitMQ Server\rabbitmq_server-3.6.10\sbin>rabbitmqctl.bat start_app
        Starting node rabbit@TWDT153
                    

        BOOT FAILED
        ===========
        
        Timeout contacting cluster nodes: ['rabbit@DESKTOP-1JHUOUJ'].
        
        BACKGROUND
        ==========
        
        This cluster node was shut down while other nodes were still running.
        To avoid losing data, you should start the other nodes first, then
        start this one. To force this node to start, first invoke
        "rabbitmqctl force_boot". If you do so, any changes made on other
        cluster nodes after this one was shut down may be lost.
        
        DIAGNOSTICS
        ===========
        
        attempted to contact: ['rabbit@DESKTOP-1JHUOUJ']
        
        rabbit@DESKTOP-1JHUOUJ:
            * unable to connect to epmd (port 4369) on DESKTOP-1JHUOUJ: nxdomain (non-exis
        ting domain)
                    
        current node details:
        - node name: rabbit@TWDT153
        - home dir: C:\Windows
        - cookie hash: OCvj/hYhPIGszubMh72YxA==
                    
        Error: timeout_waiting_for_tables
    ```

*   錯訊畫面

    ![1error](https://user-images.githubusercontent.com/3851540/29493674-73655d88-85cd-11e7-8ecb-0289d8f36079.png)

## 解決方式

1.  先停止仍在運作中的 node

    ```
    rabbitmqctl.bat stop_app
    ```

    > 如果無法順利 stop_app，可以嘗試反覆執行 `stop_app` 及 `start_app`

2.  執行 force_boot

    ```
    rabbitmqctl.bat force_boot
    ```

3.  重新啟動 RabbitMQ cluster 各個 node

    ```
    rabbitmqctl.bat start_app
    ```

    ![2forceboot](https://user-images.githubusercontent.com/3851540/29493673-73652674-85cd-11e7-9848-4654f1d2f44e.png)

## 心得

經過使用 `force_boot` 指令後，再重新啟動 RabbitMQ cluster 的各個 node 後又一切正常了，反覆執行 failover 也可正常運作，就連 node 長時間 shutdown 一樣加不回去的狀況也依舊，沒有改善 XD

說實話就是沒有根本解決問題，感覺很虛；而且解決方式也需要離線 RabbitMQ cluster 上的各個 node，這樣跟沒有 HA 是一樣的，如果有天 production 環境遇到這個狀況就完蛋了 >"<

# 參考資訊

1.  [rabbitmqctl(1) manual page](https://www.rabbitmq.com/man/rabbitmqctl.1.man.html)
2.  [RabbitMQ之鏡像隊列](http://blog.csdn.net/u013256816/article/details/71097186)
