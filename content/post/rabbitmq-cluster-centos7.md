---
title: "在 CentOS7 上建立 RabbitMQ Cluster"
date: 2021-08-02T21:30:00+08:00
lastmod: 2021-08-16T21:30:00+08:00
draft: false
tags: ["RabbitMQ","CentOS"]
slug: "rabbitmq-cluster-centos7"
---

## 在 CentOS7 上建立 RabbitMQ Cluster

新功能上線會用到 RabbitMQ，為了可以在效能與成本間取得平衡，所以打算做個效能測試，看什麼水準的硬體才能符合新功能的效能需要，在建立環境時才發現我沒有筆記過 在 CentOS7 上建立 RabbitMQ Cluster，所以趕緊補上

## 基本環境說明

1. Azure VM (CentOS 7.9 Free-Gen1;Standard_B2s) * 3

## 設定方式

1. 安裝 RabbitMQ

    > 執行對象：`每一個 node`

    ```bash
    sudo yum -y install epel-release && sudo yum update -y && sudo yum install -y rabbitmq-server
    ```

2. 建立 RabbitMQ service

    > 執行對象：`每一個 node`

    ```bash
    sudo systemctl enable rabbitmq-server
    ```

    ![1symlink](https://user-images.githubusercontent.com/3851540/127964557-20d42b47-1f02-426e-ad27-ee14ccad21e7.png)

3. 在 RabbitMQ master node 上啟動 RabbitMQ 並取得 erlang cookie

    > 執行對象：`master node`

    ```bash
    sudo systemctl start rabbitmq-server &&  sudo cat /var/lib/rabbitmq/.erlang.cookie
    ```

    ![2startandcookie](https://user-images.githubusercontent.com/3851540/127964561-614730bb-7c33-40d2-bfd9-bb7cad57a25c.png)

4. 將 RabbitMQ master node 的 erlang cookie 複製至 RabbitMQ slave 並給予 erlang cookie 正確權限

    > 執行對象：`slave node`

    ```bash
    sudo echo {master 的 erlang cookie} > /var/lib/rabbitmq/.erlang.cookie && sudo chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie && sudo chmod 400 /var/lib/rabbitmq/.erlang.cookie
    ```

    ![3setcookie](https://user-images.githubusercontent.com/3851540/127964562-6c2076ff-d1c4-4507-b560-a5a0ff5ac141.png)

5. 啟動 RabbitMQ slave node 上的 RabbitMQ

    > 執行對象：`slave node`

    ```bash
    sudo systemctl start rabbitmq-server
    ```

6. 取得 RabbitMQ master node 的 hostname

    > 執行對象：`master node`

    RabbitMQ 是透過 node name 來識別 node，而 node name 由 `{prefix}@{hostname}` 組成 ( prefix 通常為 `rabbit`)，詳細說明請參考 [Clustering Guide](https://www.rabbitmq.com/clustering.html#node-names)

    ```bash
    sudo hostname
    ```

7. 將 RabbitMQ slave node 加入 RabbitMQ master 中組成 cluster

    > 執行對象：`slave node`

    ```bash
    sudo rabbitmqctl stop_app && sudo rabbitmqctl join_cluster --ram rabbit@{hostname} && sudo rabbitmqctl start_app 
    ```

    ![4joincluster](https://user-images.githubusercontent.com/3851540/127964563-218e13b0-db45-4612-b635-15158fd79451.png)

8. 檢查 cluster 狀態

    > 執行對象：`任一 node`

    ```bash
    sudo rabbitmqctl cluster_status
    ```

    ![5clusterstatus](https://user-images.githubusercontent.com/3851540/127964565-0dc04e30-89dd-4988-96bb-c68189b0c63b.png)

## 心得

官網的文件可能是需要兼容不同 os，使用上總覺得不太順手，照著做常會卡關

雖然安裝步驟並不複雜就懶得筆記，但幾次下來還是沒能完全記得所有流程與步驟，不免東漏西漏，想了想決定花點時間筆記一下，日後自己用起來也方便些

## 參考資訊

1. [How To Setup RabbitMQ Cluster On Centos 7](https://www.unixmen.com/rabbitmq-cluster-on-centos-7/)
2. [Clustering Guide](https://www.rabbitmq.com/clustering.html#node-names)
