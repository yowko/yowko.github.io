---
title: "使用 Docker Compose 建立 Redis Cluster"
date: 2020-05-19T22:30:00+08:00
lastmod: 2020-12-11T22:30:31+08:00
draft: false
tags: ["Redis","Docker"]
slug: "docker-compose-redis-cluster"
---

## 使用 Docker Compose 建立 Redis Cluster

之前類似的主題至少有四、五篇，其中最後一版 [使用 docker 建立 Redis Cluster - 更新版](/redis-cluster-docker/) 已經滿足自己在全 container 環境中的測試需求，雖然該篇筆記下方有網友提到想要將 redis cluster 允許對外連線，當下我並沒有花時間去了解需求跟解決問題，直到最近專案需要才又調整一版

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.3.0.2(45183)

    - Engine 19.03.8
    - Compose 1.25.5

3. images

    - redis 6.0.3
    - redis 5.0.9 (測試可用)

## 設定方式

1. 專案結構

    - redis 資料夾
      - dockerfile
      - rediscluster.conf
    - docker-compose.yml
    - README.md

    ![1folder](https://user-images.githubusercontent.com/3851540/82350893-7b897300-9a2e-11ea-9343-752d89f2cfb6.jpg)

2. 建立 redis cluster node

    - dockerfile

        ```dockerfile
        FROM redis:6.0.3

        MAINTAINER Yowko Tsai <yowko@yowko.com>

        COPY rediscluster.conf /etc/redis/rediscluster.conf
        ENTRYPOINT redis-server /etc/redis/rediscluster.conf
        ```

    - rediscluster.conf

        ```config
        # ip
        bind 0.0.0.0
        # 啟用 cluster
        cluster-enabled yes
        # 指定 cluster config 檔案
        cluster-config-file nodes.conf
        # 指定 node 無法連線時間
        cluster-node-timeout 5000

        #設置主服務的連接密碼
        masterauth pass.123
        #設置從服務的連接密碼
        requirepass pass.123
        ```

3. docker-compose

    ```dock-compose
    version: '3.4'
    
    services:
      redis-node1:
        build:
          context: redis
        ports:
          - "7000:7000"
          - "17000:17000"
        restart: always
        entrypoint: [redis-server, /etc/redis/rediscluster.conf, --port,"7000", --cluster-announce-ip,"${ip}"]
      redis-node2:
        build:
          context: redis
        ports:
          - "7001:7001"
          - "17001:17001"
        restart: always
        entrypoint: [redis-server, /etc/redis/rediscluster.conf,--port,"7001",--cluster-announce-ip,"${ip}"]
      redis-node3:
        build:
          context: redis
        ports:
          - "7002:7002"
          - "17002:17002"
        restart: always
        entrypoint: [redis-server, /etc/redis/rediscluster.conf,--port,"7002",--cluster-announce-ip,"${ip}"]
      redis-node4:
        build:
          context: redis
        ports:
          - "7003:7003"
          - "17003:17003"
        restart: always
        entrypoint: [redis-server, /etc/redis/rediscluster.conf,--port,"7003",--cluster-announce-ip,"${ip}"]
        depends_on:
          - redis-node1
          - redis-node2
          - redis-node3
      redis-node5:
        build:
          context: redis
        ports:
          - "7004:7004"
          - "17004:17004"
        restart: always
        entrypoint: [redis-server, /etc/redis/rediscluster.conf,--port,"7004",--cluster-announce-ip,"${ip}"]
        depends_on:
          - redis-node1
          - redis-node2
          - redis-node3
      redis-node6:
        build:
          context: redis
        ports:
          - "7005:7005"
          - "17005:17005"
        restart: always
        entrypoint: [redis-server, /etc/redis/rediscluster.conf,--port,"7005",--cluster-announce-ip,"${ip}"]
        depends_on:
          - redis-node1
          - redis-node2
          - redis-node3
      redis-cluster-creator:
        image: redis:6.0.3
        entrypoint: [/bin/sh,-c,'echo "yes" | redis-cli -a pass.123 --cluster create ${ip}:7000 ${ip}:7001 ${ip}:7002 ${ip}:7003 ${ip}:7004 ${ip}:7005 --cluster-replicas 1']
        depends_on:
          - redis-node1
          - redis-node2
          - redis-node3
          - redis-node4
          - redis-node5
          - redis-node6
    ```

4. 使用方式

    ```bash
    ip=$(ipconfig getifaddr en0) docker-compose up -d --build
    ```

5. 實際效果

    ![2clusterinfo](https://user-images.githubusercontent.com/3851540/82350899-7cbaa000-9a2e-11ea-9a77-393618b6395c.jpg)

    ![3clusternodes](https://user-images.githubusercontent.com/3851540/82350903-7debcd00-9a2e-11ea-843b-1897799dcb0b.jpg)

## 心得

這次的調整優化項目有

1. 不需自行額外建立 network

    > ~~使用 host~~ 經網友提醒後重新 review，發現不需要

2. 不需指定每個 container ip

    > 自行分配，但 cluster node 間溝通透過 host ip 與 port

3. 不需寫死 master-slave

    > 由 redis cluster 的內部機制處理

4. 不需自行使用 dockerfile 建立三個 sentinel

    > 由 redis cluster 的內部機制處理 (之前 ip 跟 port 寫太死，需要自行控制)

5. 不需自行使用 dockerfile 來設定 cluster

    > 透過 docker compose 中的 service 來進行(之前我試不出 override entrypoint 的做法)

經過以上這些調整，就可以讓外部 redis client 連線至這次建立的 redis cluster 但還是要再次提醒這個做法<span style="color:red">僅適合用於測試</span>

除此之外，我比較喜歡這樣透過 dockerfile 搭配 docker-compose 的做法，像是 helm 或是 docker-machine 這種包裝過多，一眼很難看透的做法對我來說就像是個黑箱，你不知道裡面包了什麼、script 安不安全、是不是多執行了什麼讓效能大降之類的

完整 source code 請參考 [yowko/docker-compose-redis-cluster](https://github.com/yowko/docker-compose-redis-cluster)

## 參考資訊

1. [使用 docker 建立 Redis Cluster - 更新版](/redis-cluster-docker/)
2. [傳遞參數來執行 Docker Compose](/docker-compose-pass-parameter)
