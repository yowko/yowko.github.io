---
title: "使用 docker 建立 Redis Cluster - 更新版"
date: 2019-03-05T21:30:00+08:00
lastmod: 2019-12-19T21:30:31+08:00
draft: false
tags: ["Container","Redis","Docker"]
slug: "redis-cluster-docker"
---

## 使用 docker 建立 Redis Cluster - 更新版

之前筆記 [使用 docker 建立 Redis Cluster](https://blog.yowko.com/docker-redis-cluster) 成功建立了 redis cluster，也測試過 sentinel 可以正常 failover，興高采烈測試程式碼時才發現有 bug：在加入 cluster 時使用的是 container ip 與 port (因為 redis cluster 不支援使用 host name)，以致 redis key 在不同 slot 間無法正確做移動，所以我們馬上來看看可以如何解決吧

## 基本環境說明

1. macOS Mojave 10.14.2
2. Docker Community 18.09.2
3. docker-compose version 1.23.2, build 1110ad01
4. docker-py version: 3.6.0
5. CPython version: 3.6.6
6. OpenSSL version: OpenSSL 1.1.0h  27 Mar 2018

## 測試用程式碼

```cs
var configString= "127.0.0.1:7000,127.0.0.1:7001,127.0.0.1:7002,127.0.0.1:7003,127.0.0.1:7004,127.0.0.1:7005,password=pass.123,connectTimeout=10000,configCheckSeconds=3,allowAdmin=true";
ConfigurationOptions options = ConfigurationOptions.Parse(configString);
var conn = ConnectionMultiplexer.Connect(options);
conn.GetDatabase().StringSetAsync("test1","test123").ConfigureAwait(false).GetAwaiter().GetResult().Dump();
```

## 錯誤訊息

- 訊息內容

    ```txt
    Key has MOVED from Endpoint 172.19.0.2:6379 and hashslot 4768 but CommandFlags.NoRedirect was specified - redirect not followed for SET test1. IOCP: (Busy=0,Free=1000,Min=8,Max=1000), WORKER: (Busy=0,Free=1023,Min=8,Max=1023), Local-CPU: n/a
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/53813231-1c118580-3f98-11e9-87ad-cb2dad93c8f1.png)

## Master 與 Slave node ：內容相同

1. 準備 redis config

    > redis 預設未啟用 cluster，需要透過 config 來啟用

    ```txt
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

    ```dcokerfile
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    EXPOSE 6379 16379
    COPY rediscluster.conf /etc/redis/rediscluster.conf
    ENTRYPOINT redis-server /etc/redis/rediscluster.conf
    ```

## Sentinel node ：內容不變

1. 準備 sentinel 用的 config

    > 因為 redis cluster 有三個 node，所以 sentinel 需加入三組 replication 的 monitor

    ```config
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

    ```dockerfile
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    EXPOSE 26379
    COPY sentinel.conf /etc/redis/sentinel.conf
    ENTRYPOINT redis-server /etc/redis/sentinel.conf --sentinel
    ```

## Cluster Creator node ：內容不變

- 這邊使用固定 ip，因為 redis 使用 container name 建立 cluster 時會出現無法解析的錯誤，固定 ip 的綁定則會透過 docker-compose 處理
- 透過 redis-cli 建立 cluster 是 redis 5 加入的功能，已經無法使用 redis-trib.rb
- 建立 cluster 時需要輸入 `yes` 確認加入

    ```dockerfile
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    ENTRYPOINT echo "yes"|redis-cli -a pass.123 --cluster create 172.19.0.2:6379 172.19.0.3:6379 172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379 --cluster-replicas 1
    ```

## docker-compose ：<font style="color:red">設定改變 - 移除 port mapping</font>

> 我嘗試透過 host 做 port forward ，但在 cluster join 時一直 hang 在 wait to join 畫面，查官方文件提到除了 redis port 之外，還需要 redis port + 10000 為 redis cluster bus 使用的 port，解決方式就是直接使用 container ip 來設定 cluster

```yml
version: '3'

services:

  redis-master1:
    container_name: redis-master1
    build: dockernode
    networks:
         extnetwork:
            ipv4_address: 172.19.0.2

  redis-slave1:
    container_name: redis-slave1
    build: dockernode
    command: redis-server --slaveof redis-master1 6379
    depends_on:
      - redis-master1
    networks:
         extnetwork:
            ipv4_address: 172.19.0.3

  redis-master2:
    container_name: redis-master2
    build: dockernode
    networks:
         extnetwork:
            ipv4_address: 172.19.0.4

  redis-slave2:
    container_name: redis-slave2
    build: dockernode
    command: redis-server --slaveof redis-master2 6379
    depends_on: 
      - redis-master2
    networks:
         extnetwork:
            ipv4_address: 172.19.0.5

  redis-master3:
    container_name: redis-master3
    build: dockernode
    networks:
         extnetwork:
            ipv4_address: 172.19.0.6

  redis-slave3:
    container_name: redis-slave3
    build: dockernode
    command: redis-server --slaveof redis-master3 6379
    depends_on: 
      - redis-master3
    networks:
         extnetwork:
            ipv4_address: 172.19.0.7

  redis-cluster-creator:
    container_name: redis-cluster-creator
    build: createnode
    networks:
         extnetwork:
    depends_on:
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
    depends_on: 
      - redis-master1
      - redis-master2
      - redis-master3
      - redis-slave1
      - redis-slave2
      - redis-slave3

  redis-sentinel2:
    container_name: redis-sentinel2
    build: sentinel
    networks:
         extnetwork:
    depends_on: 
      - redis-master1
      - redis-master2
      - redis-master3
      - redis-slave1
      - redis-slave2
      - redis-slave3

  redis-sentinel3:
    container_name: redis-sentinel3
    build: sentinel
    networks:
         extnetwork:
    depends_on:
      - redis-master1
      - redis-master2
      - redis-master3
      - redis-slave1
      - redis-slave2
      - redis-slave3

networks:
   extnetwork:
      ipam:
         config:
         - subnet: 172.19.0.0/16
```

## 資料夾結構：內容相同

> 資料夾及檔案名稱只是範例，不需相同

```txt
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

## 建立  Redis Cluster 並確認運作正常：內容相同

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

## 加入 ip route rule ：<font style="color:red">新增動作</font>

讓 host 可以透過 container ip 直接 access 相關服務

- 以 Docker for Windows 為例

    > 詳細內容可以參考 [如何從 Winows docker host 透過 linux container ip 使用服務](https://blog.yowko.com/docker-host-access-linux-container-ip/)

    ```cmd
    route add 172.19.0.0 MASK 255.255.255.0 10.0.75.2
    ```

- <font style="color:red">Docker for Mac 不支援該作法</font>

## 實際使用

1. 測試用程式碼

    > redis 連線改用 container ip

    ```cs
    var configString= "172.19.0.2:6379,172.19.0.3:6379,172.19.0.4:6379,172.19.0.5:6379,172.19.0.6:6379,172.19.0.7:6379,password=pass.123,connectTimeout=10000,configCheckSeconds=3,allowAdmin=true";
    ConfigurationOptions options = ConfigurationOptions.Parse(configString);
    var conn = ConnectionMultiplexer.Connect(options);
    conn.GetDatabase().StringSetAsync("test3","test123").ConfigureAwait(false).GetAwaiter().GetResult().Dump();
    ```

2. 依 key 將資料儲存至不同 node 上

    ![2result](https://user-images.githubusercontent.com/3851540/53813233-1caa1c00-3f98-11e9-856e-f9a91115bf75.png)

## 心得

一心一意想要用 redis 5 來建立 cluster，結果反而最重要的功能沒有設定好，實在糟糕，幸虧沒有花太多時間發現問題，只是解法上依舊不甚滿意，尤其是 Docker for Mac 不支援：因為 Docker for Mac 與 Docker for Windows 及 Linux 不同，docker0 的網卡存在 vm 內部，沒辦法透過 ip route 來處理

詳細的檔案內容都在：[yowko/docker-redis-5-cluster](https://github.com/yowko/docker-redis-5-cluster.git)

## 參考資訊

1. [如何從 Winows docker host 透過 linux container ip 使用服務](https://blog.yowko.com/docker-host-access-linux-container-ip/)
2. [使用 docker 建立 Redis Cluster](https://blog.yowko.com/docker-redis-cluster)
3. [yowko/docker-redis-5-cluster](https://github.com/yowko/docker-redis-5-cluster.git)