---
title: "手動觸發 Redis failover"
date: 2020-06-07T21:30:00+08:00
lastmod: 2020-06-07T21:30:31+08:00
draft: false
tags: ["Redis"]
slug: "redis-manual-failover"
---

## 手動觸發 Redis failover

之前筆記 [手動觸發 MongoDB failover](https://blog.yowko.com/mongodb-manual-failover) 紀錄到如何透過 mongo cli 來進行 failover

今天接著紀錄如何使用 redis cli 來進行 failover

停止服務可以分為：

1. server level (服務運作的實體機器無法提供服務運行：停機 or 網路斷線)

    > 驗證方式：機器 reboot 或是拔除網路線

2. service level (服務異常：資源不足、服務本身出現錯誤....)

    > 驗證方式：停止 service process 或是 使用 service 指令執行

今天只會紀錄針對 redis service 進行 failover 的部份

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker images

    - redis 6.0.4
    - redis replication

        > 建立方式請參考之前筆記 [使用 docker 建立 Redis Master-Slave Replication Instance](https://blog.yowko.com/docker-redis-master-slave-replication/)

    - redis cluster

        > 建立方式請參考之前筆記 [使用 Helm 在 Kubernetes 上安裝 Redis Cluster](https://blog.yowko.com/mac-helm-kubernetes-redis-cluster/)

## 確認 redis master 語法

> 指令是針對 redis sentinel 的，記得連線至 redis sentinel

- 語法

    ```bash
    redis-cli [-h {redis ip}] [-p {redis port}] [-a {redis auth password}] SENTINEL get-master-addr-by-name {master name}
    ```

- 範例

    ```bash
    redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
    ```

    ![1getmaster](https://user-images.githubusercontent.com/3851540/83971312-660cb800-a90d-11ea-8ad7-f7d598d022f4.jpg)

## 在 Redis Replication 上進行 failover

> 兩種方式擇一即可，各有優缺點

1. 讓 master 暫時停止回應

    > 需要對 `master` 下指令

    - 語法

        ```bash
        redis-cli [-h {redis ip}] [-p {redis port}] [-a {redis auth password}] DEBUG sleep {秒數}
        ```

    - 範例

        ```bash
        redis-cli -p 6379 DEBUG sleep 30
        ```

    - 成功 failover

        ![2failover](https://user-images.githubusercontent.com/3851540/83971313-673de500-a90d-11ea-9bd0-1dc4152070dd.jpg)

2. sentinel api

    > 指令是針對 redis sentinel 的，記得連線至 redis sentinel

    - 語法

        ```bash
        redis-cli [-h {redis ip}] [-p {redis port}] [-a {redis auth password}] SENTINEL failover {master name}
        ```

    - 範例

        ```bash
        redis-cli -p 6379 SENTINEL failover mymaster
        ```

    - 成功 failover

        ![3sentinelapi](https://user-images.githubusercontent.com/3851540/83971314-67d67b80-a90d-11ea-8115-a56622ec1f29.jpg)

## 在 Redis Cluster 上進行 failover

> 指令需要下在 cluster replica (slave) node 上，會將該 replica 提昇為 master

1. 語法

    ```bash
    redis-cli -h {cluster replica(slave) host} -p {cluster slave port} cluster failover
    ```

2. 範例

    ```bash
    redis-cli -h redis-cluster-shard0-announce-1.redis.svc.cluster.local -p 6379 cluster failover
    ```

3. 成功 failover

    ![4clusterfailover](https://user-images.githubusercontent.com/3851540/83971315-686f1200-a90d-11ea-8cf5-f1be0d379e09.jpg)

## 心得

redis 的手動 failover 在實際執行上有較多限制：指令的對象有較嚴格的規定，但仔細想想也是合理，failover 雖然是簡單的指令，但難免對於正式服務造成異動，在指令的執行條件上有較多規範以預防人為問題出現操作錯誤是件好事

## 參考資訊

1. [使用 docker 建立 Redis Master-Slave Replication Instance](https://blog.yowko.com/docker-redis-master-slave-replication/)
2. [使用 Helm 在 Kubernetes 上安裝 Redis Cluster](https://blog.yowko.com/mac-helm-kubernetes-redis-cluster/)
3. [Redis Sentinel Documentation](https://redis.io/topics/sentinel)
4. [CLUSTER FAILOVER [FORCE|TAKEOVER]](https://redis.io/commands/cluster-failover)
