---
title: "在 CentOS 上安裝 Apache Kafka cluster"
date: 2019-11-10T21:30:00+08:00
lastmod: 2019-11-10T21:30:31+08:00
draft: false
tags: ["CentOS","Kafka"]
slug: "kafka-cluster-centos"
---

## 在 CentOS 上安裝 Apache Kafka cluster

之前筆記 [在 CentOS 上安裝 Apache ZooKeeper cluster](https://blog.yowko.com/zookeeper-cluster-centos) 紀錄到如何建立 ZooKeeper 的 cluster，雖然 ZooKeeper 還有其他用途，不過我主要打算用來與 Kafka 搭配做為 Message Queue，單看這個使用情境而言，我必需來紀錄 Kafka + Zookeeper 的安裝過程  方便日後查閱囉

## 基本環境說明

> 安裝機 X 1，實際 cluster X 3，皆透過 Azure VM 來建立

1. Linux (centos 7.7.1908)
2. Kafka 2.3.1
3. OpenJDK 11 JDK

## 安裝 JDK

> 版本需高於 JVM 8

```bash
yum install java-11-openjdk-devel
```

## 為執行 Kafka 建立專屬使用帳號

建立 user 來執行 Kafka

1. 建立 `kafka` 使用者

    > `-m` 會連帶建立 `home` 目錄： `/home/kafka`

    ```bash
    useradd kafka -m
    ```

2. 設定預設執行 shell 的程式

    ```bash
    usermod --shell /bin/bash kafka
    ```

3. 設定密碼

    ```bash
    passwd kafka
    ```

4. 讓 user `kafka` 有 `sudo` 權限

    ```bash
    usermod -aG wheel kafka
    ```

## 下載並解壓 Kafka

1. 建立下載用資料夾

    ```bash
    mkdir /home/kafka/Downloads
    ```

2. 下載 Kafka

    ```bash
    curl http://apache.stu.edu.tw/kafka/2.3.1/kafka_2.11-2.3.1.tgz -o /home/kafka/Downloads/kafka.tgz
    ```

3. 解壓縮

    ```bash
    cd /home/kafka && tar -xvzf Downloads/kafka.tgz --strip 1
    ```

4. Zookeeper 啟動設定

    - 建立啟動文件

        ```bash
        nano /etc/systemd/system/zookeeper.service
        ```

    - 啟動文件內容

        ```txt
        [Unit]
        Requires=network.target remote-fs.target
        After=network.target remote-fs.target

        [Service]
        Type=simple
        User=kafka
        ExecStart=/home/kafka/bin/zookeeper-server-start.sh /home/kafka/config/zookeeper.properties
        ExecStop=/home/kafka/bin/zookeeper-server-stop.sh
        Restart=on-abnormal

        [Install]
        WantedBy=multi-user.target
        ```

5. Kafka 啟動設定

    - 建立啟動文件

        ```bash
        vi /etc/systemd/system/kafka.service
        ```

    - 啟動文件內容

        ```txt
        [Unit]
        Requires=zookeeper.service
        After=zookeeper.service

        [Service]
        Type=simple
        User=kafka
        ExecStart=/bin/sh -c '/home/kafka/bin/kafka-server-start.sh /home/kafka/config/server.properties > /home/kafka/kafka.log 2>&1'
        ExecStop=/home/kafka/bin/kafka-server-stop.sh
        Restart=on-abnormal

        [Install]
        WantedBy=multi-user.target
        ```

6. Cluster 設定

    > 如果只需要 single node instance，可以略過這個步驟

    - Zookeeper

        > 依實際 node 來做調整

        - 調整設定

            > 位置在 `/home/kafka/config/zookeeper.properties`

            ```bash
            echo initLimit=10 >> /home/kafka/config/zookeeper.properties
            echo syncLimit=5 >> /home/kafka/config/zookeeper.properties
            echo server.1=node1:2888:3888 >> /home/kafka/config/zookeeper.properties
            echo server.2=node2:2888:3888 >> /home/kafka/config/zookeeper.properties
            echo server.3=node3:2888:3888 >> /home/kafka/config/zookeeper.properties
            ```

        - 加入 `myid` 設定  

            > 位置請參照 `/home/kafka/config/zookeeper.properties` 的 `dataDir` 設定值，每個 node 的 myid 值需要唯一

            ```bash
            echo 1 >> /tmp/zookeeper/myid
            ```

    - Kafka

        > 位置在 `/home/kafka/config/server.properties`，每個 node 都必需執行

        - 修改 `broker.id` 讓不同的 kafka instance 可以做出區隔，每個 node 的 broker.id 值需要唯一

            ```bash
            sed -i 's/broker.id=0/broker.id=node1/g' /home/kafka/config/server.properties
            ```

        - 修改 `zookeeper.connect` ，連線至 Zookeeper cluster 的位置

            ```bash
            sed -i 's/zookeeper.connect=localhost:2181/zookeeper.connect=node1:2181,node2:2181,node3:2181/g' /home/kafka/config/server.properties
            ```

## 啟動 Kafka

1. 啟動 Kafka 服務

    ```bash
    systemctl start kafka
    ```

2. 將 Kafka 設為隨開機啟動

    ```bash
    systemctl enable kafka
    ```

## 心得

有了上次安裝 ZooKeeper cluster 的經驗後，這次安裝 Kafka cluster 感覺比較上手些，不過安裝後重新看了一次步驟還是覺得相當麻煩，預計改天紀錄將上述各個步驟透過單一 shell script 來完成安裝，想必會方便許多

## 參考資訊

1. [在 CentOS 上安裝 Apache ZooKeeper cluster](https://blog.yowko.com/zookeeper-cluster-centos)
2. [How To Install Apache Kafka on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-apache-kafka-on-centos-7)
