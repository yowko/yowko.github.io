---
title: "在 CentOS 上安裝 Apache Kafka cluster - One Script"
date: 2019-11-29T21:30:00+08:00
lastmod: 2019-11-29T21:30:31+08:00
draft: false
tags: ["CentOS","Kafka"]
slug: "kafka-cluster-centos-script"
---

## 在 CentOS 上安裝 Apache Kafka cluster - One Script

之前筆記 [在CentOS 上安裝 Apache Kafka cluster](https://blog.yowko.com/kafka-cluster-centos/) 紀錄到如何在CentOS 上安裝 Apache Kafka cluster，簡單說明了有哪些設定需要做修改以及調整，但對於實際安裝的人員來說可能就太細了，所以就來紀錄一鍵全自動安裝設定完成的 script 囉

## 基本環境說明

> 安裝機 X 1，實際 cluster X 3，皆透過 Azure VM 來建立

1. Linux (centos 7.7.1908)
2. Kafka 2.3.1
3. OpenJDK 11 JDK

## 完整 Script 與使用方式

1. Script

    ```bash
    array=()

    # 將傳入參數轉為 array
    for i in "$@"; do
    echo "$i"
    array+=($i)
    done

    mkdir /tmp/kafka
    echo ">>> Downloading kafka"
    curl http://apache.stu.edu.tw/kafka/2.3.1/kafka_2.11-2.3.1.tgz -o /tmp/kafka/kafka.tgz

    zookeepers=$(IFS=, ; echo "${array[*]/%/:2181}")

    for index in "${!array[@]}"; do
        echo ">>> Install JDK"
        ssh root@${array[$index]} "yum install java-11-openjdk-devel"
        echo ">>> Copy zookeeper's daemon config"
        # 複製 service 設定 for zookeeper
        scp zookeeper.service root@${array[$index]}:/etc/systemd/system/zookeeper.service
        echo ">>> Copy kafka's daemon config"
        # 複製 service 設定 for kafka
        scp kafka.service root@${array[$index]}:/etc/systemd/system/kafka.service
        echo ">>> Copy kafka"
        # 複製 kafka 壓縮檔
        scp /tmp/kafka/kafka.tgz root@${array[$index]}:/home/kafka/kafka.tgz
        echo ">>> Unzip kafka"
        # 進入 kafka user 根路徑
        ssh root@${array[$index]} "cd /home/kafka && tar -xvzf kafka.tgz --strip 1"

        echo ">>> Config zookeeper for cluster"
        ssh root@${array[$index]} "echo initLimit=10 >> /home/kafka/config/zookeeper.properties"
        ssh root@${array[$index]} "echo syncLimit=5 >> /home/kafka/config/zookeeper.properties"
        ssh root@${array[$index]} "mkdir /tmp/zookeeper && echo \"$(($index + 1))\" > /tmp/zookeeper/myid"

        echo ">>> Setup server id for zookeeper"
        # 設定 server id for zookeeper
        for i in "${!internalArray[@]}"; do
            ssh root@${array[$index]} "echo server.\"$(($i + 1))\"=\"${internalArray[$i]}\":2888:3888 >> /home/kafka/config/zookeeper.properties"
        done

        # 修改 kafka broker_id
        echo ">>> Setup kafka broker_id"
        ssh root@${array[$index]} "sed -i 's/broker.id=0/broker.id=$(($index+1))/g' /home/kafka/config/server.properties"

        # 修改 kafka 連線的 zookeeper
        echo ">>> Setup kafka's zookeeper"
        ssh root@${array[$index]} "sed -i 's/localhost:2181/${zookeepers}/g' /home/kafka/config/server.properties"
        echo ">>> Start kafka"
        ssh root@${array[$index]} "systemctl daemon-reload && systemctl start kafka && systemctl enable kafka"
    done
    ```

2. 使用方式

    ```bash
    sh install.sh 10.0.0.5 10.0.0.6 10.0.0.7
    ```

## 心得

過去絕大多數使用的都是 Windows 平台，自從將主要 productiion os 設定為 Linux 後增加了許多寫 shell script 的機會，雖然還不到同事們的水準，但總算有在進步中，相信經過一段時間的磨練應該會好一些，現在就留個紀錄待日後重新檢視囉

## 參考資訊

1. [在CentOS 上安裝 Apache Kafka cluster](https://blog.yowko.com/kafka-cluster-centos/)
