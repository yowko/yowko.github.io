---
title: "如何在 Windows OS 安裝 Apache Kafka"
date: 2017-03-31T01:30:00+08:00
lastmod: 2018-09-16T18:26:46+08:00
draft: false
tags: ["Kafka"]
slug: "kafka-on-windows"
aliases:
    - /2017/03/windows-os-apache-kafka.html
    - /2017/03/kafka-on-windows
---
# 如何在 Windows OS 安裝 Apache Kafka
Apache Kafka 是由 LinkedIn 所開發進而 open source 的 message queue 系統，使用 scala 語言打造，具有水平擴展及高吞吐量的特性，因其效能優異而受到不少關注。

因為最近專案有 message queue 的需求所以做了幾個 message queue 的比較，最後 Kafka 憑藉著高效能表現而在網路上的獲得高人氣，也讓 RabbitMQ、ZeroMQ、ActiveMQ 等其他 queue 系統敗下陣來

在還沒實際使用前，最重要的就是建置開發環境，Kafka 當然也不例外，今天就來介紹該如何在 windows 平台中安裝 Kafka

## 必備軟體

1. Java server jre [按此下載](http://www.oracle.com/technetwork/java/javase/downloads/server-jre8-downloads-2133154.html)
2. Zookeeper [按此下載](http://zookeeper.apache.org/releases.html)
3. Kafka [按此下載](http://kafka.apache.org/downloads.html)


## 安裝 Zookeeper安裝
1. 解壓縮至硬碟 e.g. `D:\Zookeeper\`

2. 設定 config
    
    > 位置 `D:\Zookeeper\conf\`
    
    * 2-1. rename
        
        > `zoo.sample.cfg` --> `zoo.cfg`

    * 2-2. 修改 snapshot 儲存位置
        
        > `dataDir= D:\zookeeper\data`

    * 2-3. 預設使用 2181 port
        
        > `clientPort=2181`

3. 設定環境變數

    ```
    set ZOOKEEPER_HOME = D:\Zookeeper
    set PATH=%PATH%;%ZOOKEEPER_HOME%\bin;
    ```
    
    * 可以使用 `set` 列出環境變數的值
        
        > e.g. `set ZOOKEEPER_HOME`

    * 如果無法生效，請嘗試重開機

4. 啟動 zkserver
    
    > 在 command prompt 直接執行 `zkserver`

5. 成功結果

    ![1zookeeper](https://cloud.githubusercontent.com/assets/3851540/21706142/ccb9464a-d3ff-11e6-903b-81ca81d05976.png)


## 安裝 Kafka
1. 解壓縮至硬碟 
   
    > e.g. `D:\kafka\`

2. 修改設定 `server.properties`
    
    > 位置 `D:\kafka\config\server.properties`
    * 2-1. 修改 log 位置
        
        > `log.dirs=/tmp/kafka-logs` --> `d:\kafka\kafka-logs`

    *   2-2. 設定 Zookeeper 所在 server ip 及 port
        
        > `zookeeper.connect=localhost:2181`

        * 以 `,` 分隔可以指定多台
        * e.g. `127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002`

    *   2-3. 預設使用 9092 port
        *   `listeners = security_protocol://host_name:port`
        *   e.g.  `listeners = PLAINTEXT://your.host.name:9092`

3. 執行 <span style="color:red">請確認 Zookeeper 已正確啟動中</span>
    * 使用 `kafka-server-start.bat` 啟動
        * 指令 : `kafka-server-start.bat server.properties`
        *   `kafka-server-start.bat` 位於 `Kafka 安裝目錄\bin\windows\`
        *   `server.properties` 位於 `Kafka 安裝目錄\config\`
        *   e.g. `D:\kafka\bin\windows\kafka-server-start.bat D:\kafka\config\server.properties`

    * 正常執行
        
        ![2kafka](https://cloud.githubusercontent.com/assets/3851540/21706138/cc77876e-d3ff-11e6-91ca-7f08617db927.png)

## 測試檢查
1. 建立 topic
    * 使用 `kafka-topics.bat` 指令
        * 建立 `Yowkotest` topic
        * `replication factor=1` 僅有一台 Kafka
        * e.g. `"D:\kafka\bin\windows\kafka-topics.bat" --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic Yowkotest`
        * 成功建立
            ![3created](https://cloud.githubusercontent.com/assets/3851540/21706141/ccb93baa-d3ff-11e6-8f06-de4679052ed3.png)

2. 建立 Producer 及 Consumer 來測試
    * 2-1. 建立 Producer
        * `kafka-console-producer.bat --broker-list localhost:9092 --topic {topicName}`
        * e.g. `D:\kafka\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic Yowkotest`

    * 2-2. 建立 Consumer
        * `kafka-console-consumer.bat --zookeeper localhost:2181 --topic {topicName}`
        * e.g. `D:\kafka\bin\windows\kafka-console-consumer.bat --zookeeper localhost:2181 --topic Yowkotest`

3. 發送訊息
    
    ![5result](https://cloud.githubusercontent.com/assets/3851540/21706140/ccb749b2-d3ff-11e6-88db-b2fc51cf722a.png)

## 其他指令
1. 列出所有 topic
    
    ```
    kafka-topics.bat --list --zookeeper localhost:2181
    ```
2. 刪除 topic
    
    ```
    kafka-run-class.bat kafka.admin.TopicCommand --delete --topic {topicName} --zookeeper localhost:2181
    ```
    * 出現 `marked for deletion`
        ![4markeddeletion](https://cloud.githubusercontent.com/assets/3851540/21706139/cc9a9f56-d3ff-11e6-90bf-b4d1abc59ee5.png)

        * 需調整 `server.properties` 檔案，設定 `delete.topic.enable = true`

3. 讀出所有訊息
    
    ```
    kafka-console-consumer.bat --zookeeper localhost:2181 --topic {topicName} --from-beginning
    ```
    * e.g. `D:\kafka\bin\windows\kafka-console-consumer.bat --zookeeper localhost:2181 --topic Yowkotest --from-beginning`



# 參考資料
1. [Java server jre](http://www.oracle.com/technetwork/java/javase/downloads/server-jre8-downloads-2133154.html)
2. [Zookeeper](http://zookeeper.apache.org/releases.html)
3. [Kafka](http://kafka.apache.org/downloads.html)
4. [Windows OS上安裝運行Apache Kafka教程](http://geek.csdn.net/news/detail/52976)
