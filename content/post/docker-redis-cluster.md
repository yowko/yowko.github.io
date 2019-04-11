---
title: "使用 docker 建立 Redis Cluster"
date: 2019-03-02T21:30:00+08:00
lastmod: 2019-03-02T21:30:31+08:00
draft: false
tags: ["Container","Redis","Docker"]
slug: "docker-redis-cluster"
---
# 使用 docker 建立 Redis Cluster

<font style="color:red">請參考新版內容 [使用 docker 建立 Redis Cluster - 更新版](https://blog.yowko.com/redis-cluster-docker)</font>

之前筆記 [使用 docker 建立 Redis Master-Slave Replication Instance](https://blog.yowko.com/docker-redis-master-slave-replication) 紀錄到使用 docker 來建立 Redis 一個 Master node、一個 Slave node 以及三個 sentinel node 的 replication，對於一般開發需求已經綽綽有餘，但終究不是 cluster 模式，有些 cluster 的功能依舊是無法完整測試的，這時才想到過去我沒有真的在 production 環境建立過 redis cluster，主要原因是 redis replication 模式在過去的使用情境下資料使用率大多維持 30% 左右，使用 replication mode 已足夠，但隨著各系統對於 redis 的依賴，相信有天 redis replication 終究會不夠的，於是興起透過 docker 來建立 redis cluster 先做好測試準備的想法

跟 redis replication 只有一個 master node 與一個 slave node 不同，redis cluster 需有三組 master - slave，再加上三個 sentinel node，基本消費立馬來到 9 個 container

## 基本環境說明
1. macOS Mojave 10.14.2
2. Docker Community 18.09.2
3. docker-compose version 1.23.2, build 1110ad01
4. docker-py version: 3.6.0
5. CPython version: 3.6.6
6. OpenSSL version: OpenSSL 1.1.0h  27 Mar 2018

## Master 與 Slave node
1. 準備 redis config

    > redis 預設未啟用 cluster，需要透過 config 來啟用

    ```
    # 指定 redis port
    port 6379
    # 啟用 cluster
    cluster-enabled yes
    # 指定 cluster config 檔案
    cluster-config-file nodes.conf
    # 指定 node 無法連線時間
    cluster-node-timeout 5000
    # 更新操作後進行日誌記錄
    # appendonly yes
    # 設定連線至 master 密碼
    masterauth pass.123
    # 設定連線密碼
    requirepass pass.123
    ```
2. 準備 dockerfile

    > 使用上述建立的 config 來啟動 redis

    ```
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    EXPOSE 6379 16379
    COPY rediscluster.conf /etc/redis/rediscluster.conf
    ENTRYPOINT redis-server /etc/redis/rediscluster.conf
    ```


## Sentinel node
1. 準備 sentinel 用的 config

    > 因為 redis cluster 有三個 node，所以 sentinel 需加入三組 replication 的 monitor

    ```
    # sentinel port
    port 26379
    # bind ip
    bind 0.0.0.0
    
    # 監控的 redis master host 與 port，並指定兩個 sentinel 同意決定
    sentinel monitor mymaster1 redis-master1 6379 2
    # 無法連線 3000 毫秒，判定為離線
    sentinel down-after-milliseconds mymaster1 3000
    # 同時可以從 master 拉取資料的 slave 個數為 1
    sentinel parallel-syncs mymaster1 1
    # sentinel 執行 failover 失敗時間為 10000 毫秒
    sentinel failover-timeout mymaster1 10000

    sentinel monitor mymaster2 redis-master2 6379 2
    sentinel down-after-milliseconds mymaster2 3000
    sentinel parallel-syncs mymaster2 1
    sentinel failover-timeout mymaster2 10000

    sentinel monitor mymaster3 redis-master3 6379 2
    sentinel down-after-milliseconds mymaster3 3000
    sentinel parallel-syncs mymaster3 1
    sentinel failover-timeout mymaster3 10000
    ```
2. 準備 dockerfile

    > 使用上述的 sentinel config 來啟動 sentinel

    ```
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    EXPOSE 26379
    COPY sentinel.conf /etc/redis/sentinel.conf
    ENTRYPOINT redis-server /etc/redis/sentinel.conf --sentinel
    ```

## Cluster Creator node

- 這邊使用固定 ip，因為 redis 使用 container name 建立 cluster 時會出現無法解析的錯誤，固定 ip 的綁定則會透過 docker-compose 處理
- 透過 redis-cli 建立 cluster 是 redis 5 加入的功能，已經無法使用 redis-trib.rb
- 建立 cluster 時需要輸入 `yes` 確認加入

    ```
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    ENTRYPOINT echo "yes"|redis-cli -a pass.123 --cluster create 172.19.0.2:6379 172.19.0.3:6379 172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379 --cluster-replicas 1
    ```

## docker-compose

> 我嘗試透過 host 做 port forward ，但在 cluster join 時一直 hang 在 wait to join 畫面，查官方文件提到除了 redis port 之外，還需要 redis port + 10000 為 redis cluster bus 使用的 port，解決方式就是直接使用 container ip 來設定 cluster

```yml
version: '3'

services:

  redis-master1:
    container_name: redis-master1
    build: dockernode
    ports: 
      - "7000:6379"
    networks:
         extnetwork:
            ipv4_address: 172.19.0.2

  redis-slave1:
    container_name: redis-slave1
    build: dockernode
    command: redis-server --slaveof redis-master1 6379
    links:
      - redis-master1
    ports: 
      - "7001:6379"
    networks:
         extnetwork:
            ipv4_address: 172.19.0.3

  redis-master2:
    container_name: redis-master2
    build: dockernode
    ports: 
      - "7002:6379"
    networks:
         extnetwork:
            ipv4_address: 172.19.0.4

  redis-slave2:
    container_name: redis-slave2
    build: dockernode
    command: redis-server --slaveof redis-master2 6379
    links:
      - redis-master2
    ports: 
      - "7003:6379"
    networks:
         extnetwork:
            ipv4_address: 172.19.0.5

  redis-master3:
    container_name: redis-master3
    build: dockernode
    ports: 
      - "7004:6379"
    networks:
         extnetwork:
            ipv4_address: 172.19.0.6

  redis-slave3:
    container_name: redis-slave3
    build: dockernode
    command: redis-server --slaveof redis-master3 6379
    links:
      - redis-master3
    ports: 
      - "7005:6379"
    networks:
         extnetwork:
            ipv4_address: 172.19.0.7

  redis-cluster-creator:
    container_name: redis-cluster-creator
    build: createnode
    networks:
         extnetwork:
    links:
      - redis-master1
      - redis-master2
      - redis-master3
      - redis-slave1
      - redis-slave2
      - redis-slave3

  redis-sentinel1:
    container_name: redis-sentinel1
    build: sentinel
    networks:
         extnetwork:
    links:
      - redis-master1
      - redis-master2
      - redis-master3

  redis-sentinel2:
    container_name: redis-sentinel2
    build: sentinel
    networks:
         extnetwork:
    links:
      - redis-master1
      - redis-master2
      - redis-master3

  redis-sentinel3:
    container_name: redis-sentinel3
    build: sentinel
    networks:
         extnetwork:
    links:
      - redis-master1
      - redis-master2
      - redis-master3

networks:
   extnetwork:
      ipam:
         config:
         - subnet: 172.19.0.0/16
```

## 資料夾結構

> 資料夾及檔案名稱只是範例，不需相同

```
-- sentinel
    -- sentinel.conf
    -- dockerfile
-- dockernode
    -- rediscluster.conf
    -- dockerfile
-- createnode
    -- dockerfile
-- docker-compose.yml
```

![1folderstructure](https://user-images.githubusercontent.com/3851540/53691577-dbf5ab80-3dbb-11e9-88e4-8dbe0f922bb0.png)


## 建立  Redis Cluster 並確認運作正常

1. 進入 `docker-redis-cluster` 資料夾中

    ```bash
    cd docker-redis-cluster
    ```

2. 啟動 Redis Cluster


    ```bash
    docker-compose up -d --build
    ```

3. 確認 Redis Cluster 正確設定

    > 出現 `cluster_state:ok` 即是正確設定

    ```bash
    docker exec -it redis-master1 redis-cli -a pass.123 -c cluster info
    ```

    ![2clusterinfo](https://user-images.githubusercontent.com/3851540/53691578-dbf5ab80-3dbb-11e9-90af-026926abc619.png)

## 心得
查了些資料，發現網路上多數透過 docker 建立 redis cluster 的例子都是使用 redis 4 而不是 redis 5，我猜測是因為 redis 5 預設使用 redis-cli 而無法使用 ruby 或是 python 來建立 cluster，讓原本可以從 shell script 中處理掉 container ip 取得與使用而不需指定 ip 的做法失效

原本我還不信邪地把想到的方法試過一輪，確認 redis 5 建立 cluster 暫時沒有比較漂亮的做法了, 不過如果目的只是測試應該夠用了，需要留意的只有 container ip 不要與 host 環境衝突即可

詳細的檔案內容都在：[yowko/docker-redis-5-cluster](https://github.com/yowko/docker-redis-5-cluster.git)

<font style="color:red">請參考新版內容 [使用 docker 建立 Redis Cluster - 更新版](https://blog.yowko.com/redis-cluster-docker)</font>

# 參考資訊
1. [使用 docker 建立 Redis Master-Slave Replication Instance](https://blog.yowko.com/docker-redis-master-slave-replication)
2. [Grokzen/docker-redis-cluster](https://github.com/Grokzen/docker-redis-cluster)
3. [How to answer install question in dockerfile?](https://stackoverflow.com/questions/52257866/how-to-answer-install-question-in-dockerfile)
4. [docker-compose自定義網絡，固定容器ip地址](https://blog.csdn.net/hechaojie_com/article/details/83625265)
5. [Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)