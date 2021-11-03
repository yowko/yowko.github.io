---
title: "使用 docker 建立 Redis Master-Slave Replication Instance"
date: 2019-02-28T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["Container","Redis","Docker"]
slug: "docker-redis-master-slave-replication"
---
## 使用 docker 建立 Redis Master-Slave Replication Instance

同事問到 StackExchange.Redis 的相關功能，首先必要條件就是建立測試環境，測試環境有大有小：簡易功能，單個 node 的 redis 絕對可以滿足大部份需求，但如果是想要測試 High Availability 完整功能最低消費就是 Redis Master-Slave Replication：需要一個 Master 、一個 Slave 跟至少三個 Sentinel，只是偶爾建立完整環境還可以當做複習相關設定，如果是在有時間壓力或當下只想專注測試程式面問題，不想多花時間在搞這些設定，所以興起用 docker 快速建立 Redis Master-Slave Replication Instance 的念頭，立馬來看看可以如何設定

## 基本環境說明

1. macOS Mojave 10.14.2
2. Docker Community 18.09.2
3. docker-compose version 1.23.2, build 1110ad01
4. docker-py version: 3.6.0
5. CPython version: 3.6.6
6. OpenSSL version: OpenSSL 1.1.0h  27 Mar 2018

## 建立 Sentinel image

1. sentinel.conf

    ```config
    # sentinel port
    port 26379
    # 因為 bind ip 相對不好處理，暫時先關閉 protected mode
    protected-mode no
    # 監控的 redis master host 與 port，並指定兩個 sentinel 同意決定
    sentinel monitor mymaster redis-master 6379 2
    # 無法連線 3000 毫秒，判定為離線
    sentinel down-after-milliseconds mymaster 3000
    # 同時可以從 master 拉取資料的 slave 個數為 1
    sentinel parallel-syncs mymaster 1
    # sentinel 執行 failover 失敗時間為 10000 毫秒
    sentinel failover-timeout mymaster 10000
    ```

2. sentinel 的 dockerfile

    ```dockerfile
    # 使用 base image 為 alpine 3.9 且 redis 為 5.0.3 的版本
    FROM redis:5.0.3-alpine3.9

    MAINTAINER Yowko Tsai <yowko@yowko.com>

    # 對外使用 26379
    EXPOSE 26379
    # 將 sentinel.conf 複製至 container 的 /etc/redis/sentinel.conf
    COPY sentinel.conf /etc/redis/sentinel.conf
    # container 啟動時執行的指令
    ENTRYPOINT redis-server /etc/redis/sentinel.conf --sentinel
    ```

## Redis Master-Slave Replication Instance dcoker-compose

```yml
version: '3'

services:

  redis-master:
    container_name: redis-master
    image: redis:5.0.3-alpine3.9
    ports:
      - "6379:6379"

  redis-slave:
    container_name: redis-slave
    image: redis:5.0.3-alpine3.9
    command: redis-server --slaveof redis-master 6379
    links:
      - redis-master
    ports:
      - "6380:6379"

  redis-sentinel-1:
    container_name: redis-sentinel-1
    build: sentinel
    links:
      - redis-master

  redis-sentinel-2:
    container_name: redis-sentinel-2
    build: sentinel
    links:
      - redis-master

  redis-sentinel-3:
    container_name: redis-sentinel-3
    build: sentinel
    links:
      - redis-master
```

## 檔案位置資料結構

```txt
-- docker-replication 
   -- docker-compose.yml
   -- sentinel
      -- dockerfile
      -- sentinel.conf 
```

![1folderstructure](https://user-images.githubusercontent.com/3851540/53646941-684a8600-3c77-11e9-8fed-2d4642127dde.png)

## 實際使用

1. 進入至存放 docker-compose 資料夾中

    ```bash
    cd docker-replication
    ```

2. 建立 docker replication instance

    ```bash
    docker-compose up -d --build
    ```

    ![2dockerup](https://user-images.githubusercontent.com/3851540/53646942-684a8600-3c77-11e9-952e-a513ae3baacf.png)
3. 確認 master

    ```bash
    docker exec -it redis-master redis-cli info | grep 'role'
    ```

    ![3masterrole](https://user-images.githubusercontent.com/3851540/53646943-684a8600-3c77-11e9-96a2-f95bf96eedd6.png)

4. 確認 slave

    ```bash
    docker exec -it redis-slave redis-cli info | grep 'role'
    ```

    ![4slaverole](https://user-images.githubusercontent.com/3851540/53646944-68e31c80-3c77-11e9-8fd0-147d287a0903.png)

5. 觸發 failover

    ```bash
    docker stop redis-master
    ```

6. slave 已轉為 master

    ![5failover](https://user-images.githubusercontent.com/3851540/53646945-68e31c80-3c77-11e9-8777-a4ff6ac1a713.png)

7. 重新啟動 failover 停掉的 master

    ```bash
    docker start redis-master
    ```

8. 確認重新啟動的原 master 已轉為 slave

    ![6masterrecovery](https://user-images.githubusercontent.com/3851540/53646946-68e31c80-3c77-11e9-9961-73a19462d862.png)

* 移除整個 instance

    > 停止並移除所有 container 及網路，手動移除 container 容易忽略 network

    ```bash
    docker-compose down
    ```

    ![7dockercomposedown](https://user-images.githubusercontent.com/3851540/53646948-697bb300-3c77-11e9-8bf8-cb31e04548ed.png)

## 心得

透過三個檔案 (sentinel.conf,dockerfile,docker-compose.yml)，一個指令 (docker-compose up -d --build) 就可以建立完整功能的 Redis Replication instance，實在有夠方便的啦，想起以前傻傻地建立每個 node 真的很辛苦呀，不過也是因為過去實際的操作經驗才知道可以如何利用 docker 來簡化流程，而不是針對網路資料所有內容照單全收，果然過去的努力都是累積實力的一部份不會付諸流水

docker-compose 及 dockerfile 完整內容可以參考 [yowko/redis5-replication](https://github.com/yowko/redis5-replication)

## 參考資訊

1. [每天5分鐘玩轉 OpenStack](https://www.ibm.com/developerworks/community/blogs/132cfa78-44b0-4376-85d0-d3096cd30d3f/entry/RUN_vs_CMD_vs_ENTRYPOINT_%E6%AF%8F%E5%A4%A95%E5%88%86%E9%92%9F%E7%8E%A9%E8%BD%AC_Docker_%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF_17?lang=en)
2. [itsmetommy/docker-redis4](https://github.com/itsmetommy/docker-redis4)
