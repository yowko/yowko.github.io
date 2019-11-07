---
title: "在 CentOS 上安裝 Apache ZooKeeper cluster"
date: 2019-11-02T21:30:00+08:00
lastmod: 2019-11-02T21:30:31+08:00
draft: false
tags: ["CentOS","ZooKeeper"]
slug: "zookeeper-cluster-centos"
---

## 在 CentOS 上安裝 Apache ZooKeeper cluster

之前開發時都透過 dcoker 來啟動，最近才猛然想起我沒有實際安裝過 Apache ZooKeeper cluster，趕緊趁著假日空閒時間惡補一下 Apache ZooKeeper cluster 安裝

只是小弟既不熟 Linux 也不是 Apache ZooKeeper cluster 的熟手，不正確的部份還有勞各位大大指教了

## 基本環境說明

1. CentOS Linux release 7.6.1810 (Core)
2. ZooKeeper 3.5.6

    > 版本可以自行從 [ZooKeeper Releases](https://archive.apache.org/dist/zookeeper/) 挑選

3. OpenJDK 11 JDK

## 安裝 Java

```bash
yum install java-11-openjdk-devel
```

## 建立 Zookeeper 用的 user

建立 user 來執行 Zookeeper

1. 建立 `zookeeper` 使用者

    > `-m` 會連帶建立 `home` 目錄： `/home/zookeeper`

    ```bash
    useradd zookeeper -m
    ```

2. 設定預設執行 shell 的程式

    ```bash
    usermod --shell /bin/bash zookeeper
    ```

3. 設定密碼

    ```bash
    passwd zookeeper
    ```

4. 讓 user `zookeeper` 有 `sudo` 權限

    ```bash
    usermod -aG wheel zookeeper
    ```

## 建立 Zookeeper 用資料夾

1. 建立儲存 config 與資料的資料夾

    ```bash
    mkdir -p /data/zookeeper
    ```

2. 設定資料夾權限

    ```bash
    chown -R zookeeper:zookeeper /data/zookeeper
    ```

## 下載並解壓 ZooKeeper

1. 在 `/opt` 路徑中，下載 ZooKeeper 3.5.6

    > 想選用其他版本可以看 [ZooKeeper Releases](https://archive.apache.org/dist/zookeeper/)

    ```bash
    cd /opt
    curl https://archive.apache.org/dist/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6-bin.tar.gz -o zookeeper.tar.gz
    ```

2. 解壓 `zookeeper`

    ```bash
    tar -xvf zookeeper.tar.gz
    ```

3. 換掉原始資料夾名稱

    ```bash
    mv apache-zookeeper-3.5.6-bin zookeeper
    ```

4. 為 user `zookeeper` 加上執行權限

    ```bash
    chown -R zookeeper:zookeeper /opt/zookeeper
    ```

## 設定 ZooKeeper

1. 在 `/opt/zookeeper/conf` 中建立 `zoo.cfg`

    ```bash
    nano /opt/zookeeper/conf/zoo.cfg
    ```

2. 加入設定值

    ```txt
    tickTime=2000
    dataDir=/data/zookeeper
    clientPort=2181
    maxClientCnxns=60
    ```

## 啟動 ZooKeeper 並測試單一安裝

1. 啟動

    ```bash
    /opt/zookeeper/bin/zkServer.sh start
    ```

    > 成功啟動

    ![1startzookeeper](https://user-images.githubusercontent.com/3851540/68084517-a553c180-fe71-11e9-8132-2f21b310c564.png)

2. 測試連線

    ```bash
    /opt/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181
    ```

    > 順利連線

    ![2connected](https://user-images.githubusercontent.com/3851540/68084518-a5ec5800-fe71-11e9-876f-4c215025b5e3.png)

3. 停止

    ```bash
    /opt/zookeeper/bin/zkServer.sh stop
    ```

## 將 ZooKeeper 設為為服務

1. 新增服務設定

    ```bash
    nano /etc/systemd/system/zookeeper.service
    ```

2. 加入設定內容

    > 這邊安裝時一直遇到 `zookeeper` 有權限問題，暫時先改用 `root` 執行，之後再查

    ```bash
    [Unit]
    Description=Zookeeper Daemon
    Documentation=http://zookeeper.apache.org
    Requires=network.target
    After=network.target

    [Service]
    Type=forking
    WorkingDirectory=/opt/zookeeper
    User=root
    ExecStart=/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg
    ExecStop=/opt/zookeeper/bin/zkServer.sh stop /opt/zookeeper/conf/zoo.cfg
    ExecReload=/opt/zookeeper/bin/zkServer.sh restart /opt/zookeeper/conf/zoo.cfg
    TimeoutSec=30
    Restart=on-failure

    [Install]
    WantedBy=default.target
    ```

3. 啟動服務

    ```bash
    systemctl start zookeeper
    ```

4. 設定開機預設啟動

    ```bash
    systemctl enable zookeeper
    ```

## 設定 ZooKeeper cluster

> 在 ZooKeeper cluster 每個 node 執行上述每個步驟


1. 修改 ZooKeeper 設定值

    ```bash
    nano /opt/zookeeper/conf/zoo.cfg
    ```

2. 設定內容

    ```txt
    tickTime=2000
    dataDir=/data/zookeeper
    clientPort=2181
    maxClientCnxns=60
    initLimit=10
    syncLimit=5
    server.1=node1:2888:3888
    server.2=node2:2888:3888
    server.3=node3:2888:3888
    ```

3. 設定 Zookeeper 識別用 id

    >每個 node 都不同

    - node1

        ```bash
        echo "1" > /data/zookeeper/myid
        ```

    - node2

        ```bash
        echo "2" > /data/zookeeper/myid
        ```

    - node3

        ```bash
        echo "3" > /data/zookeeper/myid
        ```

4. 重新啟動每個 node 上的 Zookeeper

    ```bash
    systemctl restart zookeeper
    ```

5. 測試實際效果

    - 連線至任一 Zookeeper node 中

        ```bash
        /opt/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181
        ```

    - 建立資料

        ```bash
        create /yowkotest
        ```

    - 至其他 node 中確認是否可以看到新增的 topic

        ```bash
        /opt/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181
        ls /
        ```

## 心得

安裝前我還以為很簡單，但實際上眉眉角角比想的還多不少，不知道是我對 Linux 的權限系統不熟悉造成的，還是 Linux 的安裝就是這樣

就以安裝的步驟而言，我覺得並不太直覺，這些動作叫我不看文件我一定無法完成，不過先求有再求好，有機會再來改善囉

此外，現在是不是 Ubuntu 比較受歡迎呀？ 網路上的 blog 較新資料都是 Ubuntu 上的安裝，CentOS 並不多，讓我只好自己踩雷了

## 參考資訊

1. [ZooKeeper Releases](https://archive.apache.org/dist/zookeeper/)
2. [How to install Java on CentOS 7](https://linuxize.com/post/install-java-on-centos-7/)
3. [How To Install and Configure an Apache ZooKeeper Cluster on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-an-apache-zookeeper-cluster-on-ubuntu-18-04)
4. [How to Setup Apache ZooKeeper Cluster on Ubuntu 18.04 LTS](https://www.howtoforge.com/tutorial/how-to-setup-apache-zookeeper-cluster-on-ubuntu-1804/)
5. [Zookeeper 3.2 叢集安裝](https://sites.google.com/site/waue0920/Home/zookeeper/zookerper-an-zhuang-cong-ji)
