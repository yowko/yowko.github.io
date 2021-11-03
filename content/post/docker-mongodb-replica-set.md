---
title: "使用 docker 建立 MongoDB Replica Set"
date: 2019-03-07T01:30:00+08:00
lastmod: 2021-11-03T01:30:31+08:00
draft: false
tags: ["Container","MongoDB","Docker"]
slug: "docker-mongodb-replica-set"
---
## 使用 docker 建立 MongoDB Replica Set

前幾天筆記 [使用 docker 建立 Redis Cluster - 更新版](/redis-cluster-docker) 提到為了要測試 Redis 完整 cluster 功能但又不想每次都重頭建立三組 Master-Slave 以及三個 Sentinel，所以透過 docker-compose 來建立整個 Redis cluster，而最近用到的 MongoDB Replica Set 也是類似情況，但比 Redis 輕鬆的是 MongoDB Replica Set 只要三個 node 即可，就來看看可以怎麼做吧

## 基本環境說明

- Mac
    1. macOS Mojave 10.14.2
    2. Docker Community 18.09.2
    3. docker-compose version 1.23.2, build 1110ad01
    4. docker-py version: 3.6.0
    5. CPython version: 3.6.6
    6. OpenSSL version: OpenSSL 1.1.0h  27 Mar 2018
- Wondows
    1. Windows 10 Version 1803 (OS Build 17134.590)
    2. Docker Community 18.09.2 (linux/amd64)

## 建立 dockerfile

> 這個 image 是用來設定 MongoDB 的 Replica Set 前讓三個 MongoDB 暖機用的，實際上沒什麼作用，只是我找不到可以在 docker-compse 中同時執行 sleep 與 mongo shell 的方法而做的

```dockerfile
FROM mongo:4-xenial

MAINTAINER Yowko Tsai <yowko@yowko.com>

CMD ["sleep","10"]
```

## 建立 docker-compose.yml

```yml
version: "3"
services:
  mongo1:
    container_name: mongo1
    image: mongo:4-xenial
    expose:
      - 27017
    restart: always
    entrypoint: [ "mongod", "--bind_ip_all", "--replSet", "rs0" ]
  mongo2:
    container_name: mongo2
    image: mongo:4-xenial
    expose:
      - 27017
    restart: always
    entrypoint: [ "mongod", "--bind_ip_all", "--replSet", "rs0" ]
  mongo3:
    container_name: mongo3
    image: mongo:4-xenial
    expose:
      - 27017
    restart: always
    entrypoint: [ "mongod", "--bind_ip_all", "--replSet", "rs0" ]
  creator:
    build: creator
    entrypoint: ["mongo","--host","mongo1","--port","27017","--eval", 'rs.initiate( { _id : "rs0",members: [{ _id: 0, host: "mongo1:27017" },{ _id: 1, host: "mongo2:27017" },{ _id: 2, host: "mongo3:27017" }   ]})']
    depends_on:
      - mongo1
      - mongo2
      - mongo3
```

## 資料夾結構

```txt
-- docker-mongodb-replica-set
    -- creator
        -- dockerfile
    -- docker-compose.yml
```

![folderstructure](https://user-images.githubusercontent.com/3851540/53899681-b72c5d00-4075-11e9-9558-e5cc2e804a29.png)

## 啟動與實際運作

1. 啟動建立 MongoDB Replica Set

    ```bash
    docker-compose up -d --build
    ```

    ![_output_2dockerup](https://user-images.githubusercontent.com/3851540/53900425-7897a200-4077-11e9-8ba1-214065b2b535.png)

2. 針對 mongo1 執行連線 mongo 連線至預設 localhost:27017 的 MongoDB

    ```bash
    docker exec -it mongo1 mongo
    ```

    ![_output_3connectmongo](https://user-images.githubusercontent.com/3851540/53900426-7897a200-4077-11e9-8cd5-a182035cc996.png)

3. 確認 Replica Set 狀態

    ```js
    rs.status()
    ```

    ![_output_4rsstatus](https://user-images.githubusercontent.com/3851540/53900427-79303880-4077-11e9-9ea4-9e9ad5798248.png)

## 心得

自己動手做之前我有 google 過其他人做法，大部份都是透過 shell 來加入 Replica Set，會多一個 conatiner，加上 shell 又是另種語法，我個人不想額外 maintain shell 所以才想利用 docker 相關功能解決就好

以結果來看的確是完成的 MongoDB Replica Set 的建立，但多了個沒實際作用的 container ，作法上不是很漂亮，不過先求有，如果之後被嫌棄或是緣份到了再來找更好的做法囉

完整 dockerfile 及 docker-compose.yml 請參考 [yowko/Docker-Compose-MongoDB-Replica-Set](https://github.com/yowko/Docker-Compose-MongoDB-Replica-Set)

## 參考資訊

1. [MongoDB 在 Windows 上的 HA 機制 - Replica Sets](/mongodb-windows-ha-replica-sets/)
2. [How to execute mongo commands through shell scripts?](https://stackoverflow.com/a/6000360)
3. [yowko/Docker-Compose-MongoDB-Replica-Set](https://github.com/yowko/Docker-Compose-MongoDB-Replica-Set)
