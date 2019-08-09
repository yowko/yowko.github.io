---
title: "建立 Redis Cluster (Redis 5)"
date: 2019-08-09T18:30:00+08:00
lastmod: 2019-08-09T18:30:31+08:00
draft: false
tags: ["Redis"]
slug: "create-redis-cluster"
---

## 建立 Redis Cluster (Redis 5)

最近在部署上線用的各項服務，其中一個就是近幾年系統中不可獲缺的 redis，目前團隊打算採用 Redis Cluster 來增加 redis 處理效能及水平擴展能力，只是對於 redis cluster 我只有自己測試的經驗沒有真正應用在 production 過，重新檢視文件時發現 Redis 5 有新的 cluster 架設方式，測試順便筆記一下

## 基本環境說明

1. macOS Mojave 10.14.6
2. Redis 5.0.5

## 關於 Redis Cluster

1. 僅適合於 Redis3.0（包括3.0）以上版本
2. 每個 redis cluster 有 16384 個 hash slot，key 經過 CRC16 計算後決定儲存的 cluster node
3. 每個 redis node 都需要開放兩個 TCP port：一個是 redis client 連線使用的 port (常見為 `6379`) 另一個是 cluster bus 用來做 node 間的交互溝通 (固定為 client port 加上 `10000`)
4. 僅支援單一 database
5. 不支援多個 key 的操作 (可能需要跨 node)
6. 最少需要三個 master node，但建議使用 3 個 master nodes + 3 個 slave node

## 下載 Redis 並編譯

[官網](https://redis.io/download)即提供完整 source code 下載，以下使用 `Stable` 版本

```bash
curl -O http://download.redis.io/releases/redis-5.0.5.tar.gz
tar xzf redis-5.0.5.tar.gz
cd redis-5.0.5
make
```

## 使用自訂 config

1. 準備 conffig

    > 使用 6 個 node (3 master + 3 slave)：7000,7001,7002,7003,7004,7005，因此需要 6 個 config，如果是同一台電腦上使用不同 port 來建立 instance ，記得 config 中的 `port` 與 `cluster-config-file` 要切分開來

    - 最基本的 cluster node config

        ```conf
        port 7000 # 每個 node 的 client port
        cluster-enabled yes # 啟用 redis cluster
        cluster-config-file nodes_7000.conf # 每個 node 需要獨立，cluster 自行維護使用，不需人為介入
        cluster-node-timeout 5000 # node 判斷失效的時間
        appendonly yes # 啟用 aof
        ```

    - 其他可能需要的設定

        ```conf
        port 7000 # 每個 node 的 client port
        cluster-enabled yes # 啟用 redis cluster
        cluster-config-file nodes_7000.conf # 每個 node 需要獨立，cluster 自行維護使用，不需人為介入
        cluster-node-timeout 5000 # node 判斷失效的時間
        appendonly yes # 啟用 aof
        daemonize yes # 背景執行
        bind x.x.x.x # 允許 listen 特定 ip 的連線
        requirepass password # 密碼設定
        masterauth password # 從 master 的密碼
        ```

        ![1configpath](https://user-images.githubusercontent.com/3851540/62774852-a7a7b900-bad8-11e9-83ee-b52ca8c3123d.png)

2. 啟動

    使用之前 build 出來的 redis-server.sh 與準備好的 config 逐一啟動 redis instance

    ```bash
     ./redis-5.0.5/src/redis-server ./7000/redis.conf
     ./redis-5.0.5/src/redis-server ./7001/redis.conf
     ./redis-5.0.5/src/redis-server ./7002/redis.conf
     ./redis-5.0.5/src/redis-server ./7003/redis.conf
     ./redis-5.0.5/src/redis-server ./7004/redis.conf
     ./redis-5.0.5/src/redis-server ./7005/redis.conf
    ```

3. 加入 cluster

    ```bash
    redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
    ```

    ![2clusterok](https://user-images.githubusercontent.com/3851540/62774853-a8404f80-bad8-11e9-9b92-67fc0c80d5af.png)

4. 取得 cluster 資訊

    > 可以對 cluster 任一 node 執行指令

    - cluster nodes

        > 可以確認 cluster 現在的 node、每個 node 的 role 以及分配的 slot

        ```bash
        ./redis-5.0.5/src/redis-cli -p 7000 cluster nodes
        ```

        ![3clusternode](https://user-images.githubusercontent.com/3851540/62774854-a8404f80-bad8-11e9-96a1-7852f216fe94.png)

    - cluster info

        ```bash
        ./redis-5.0.5/src/redis-cli -c -p 7000 cluster info
        ```

        > 可以確認 cluster 的狀態

        ![4clusterinfo](https://user-images.githubusercontent.com/3851540/62774855-a8404f80-bad8-11e9-9abb-8ecf4b3eb956.png)

## 使用內建 script

如果主要的測試是放在 client 端，純粹只是想要有個 redis cluster 提供服務，照上面方式逐一建立 config 、啟動 redis 再將所有 redis instance join 為 cluster 的動作就變得有些擾人，這時就可以使用內建的 shell 來快速建立 redis cluster

需要在  `./utils/create-cluster/create-cluster.sh` 所在目錄下執行 `create-cluster.sh` 才不會遇到路徑問題

1. 啟動 redis instance (3 master + 3 slave)

    > 預設會建立 30001,30002,30003,30004,30005,30006 六個 redis instance

    ```bash
    ./create-cluster start
    ```

    ![5clusterstart](https://user-images.githubusercontent.com/3851540/62774856-a8404f80-bad8-11e9-8d6d-99bfb2fd2335.png)

2. 建立 redis cluster

    ```bash
    ./create-cluster create
    ```

    ![6clustercreate](https://user-images.githubusercontent.com/3851540/62774857-a8d8e600-bad8-11e9-86b7-9ef26916b008.png)

3. 停止 redis cluster

    ```bash
    ./create-cluster stop
    ```

    ![7cluserstop](https://user-images.githubusercontent.com/3851540/62774859-a8d8e600-bad8-11e9-8d77-0e392e3f5f44.png)

4. 完整清除 redis cluster 相關設定

    ```bash
    ./create-cluster clean
    ```

## 心得

測試下來 redis 5 在 cli 上對於 redis cluster 的整合更完整了，過去只能透過其他 shell 來建立 cluster，現在不需要再想辦法看懂這些其他語言的 script 即可建立，相對方便許多，不過卻也有可能值得改進的地方：master 與 slave 的分配都是 cli 決定，反而失去了一些彈性，我嘗試在某些 node 的 config 上加入 `replicaof` 會無法正確啟動 redis

![8error](https://user-images.githubusercontent.com/3851540/62774860-a8d8e600-bad8-11e9-9764-46d79f7af220.png)

另外 redis-cli 在建立 redis cluster 時也將 sentinel 機制加入，也省去了自己建立 sentinel 時間跟精力

## 參考資訊

1. [Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)
