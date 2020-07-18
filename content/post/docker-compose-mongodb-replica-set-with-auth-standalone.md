---
title: "使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set (Single Node)"
date: 2020-07-11T21:30:00+08:00
lastmod: 2020-07-11T21:30:31+08:00
draft: false
tags: ["MongoDB","Docker"]
slug: "docker-compose-mongodb-replica-set-with-auth-standalone"
---

## 使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set (Single Node)

之前在筆記 [Docker Compose 建立 MongoDB Replica Set](https://blog.yowko.com/docker-compose-mongodb-replica-set/) 紀錄到使用 Docker Compose 建立 MongoDB Replica Set，不過當時建立的 MongoDB Replica Set 主要目的是為了驗證資料同步以及讀寫分離的功能並沒有加上 auth 相關機制

雖說 docker compose 建立的 MongoDB Replica Set 本來就只適合用在開發測試階段，對於安全性要求不那麼講究，只是沒有 auth 機制也就表示 auth 功能沒驗證到，難保實際上線後 auth 機制才出現問題，所以趁著假日空閒時間試試做法

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker images

    - mongo:4.2.8-bionic

## 建立方式

1. docker-compose.yaml

    ```yaml
    version: "3.4"

    services:
      mongo1:
        container_name: mongo1
        image: mongo:4.2.8-bionic
        environment:
          - MONGO_INITDB_ROOT_USERNAME=root
          - MONGO_INITDB_ROOT_PASSWORD=password
        ports:
          - 27017:27017
        healthcheck:
          test:
            [
              "CMD",
              "mongo",
              "admin",
              "-u",
              "root",
              "-p",
              "password",
              "--eval",
              'rs.initiate( { _id : "rs0",members: [{ _id: 0,host: "mongo1:27017" }]}); db.getSiblingDB("admin").grantRolesToUser("root",[ "clusterAdmin" ]);',
            ]
          interval: 10s
          start_period: 15s
        command: ["--replSet", "rs0", "--bind_ip_all", "--auth"]
    ```

2. 啟動 mongo

    ```bash
    docker-compose up -d
    ```

3. 實際使用

    - 沒有提供 auth，無法取得 replica set 資訊

        ![1noauth](https://user-images.githubusercontent.com/3851540/87247548-ff883600-c486-11ea-85a4-72419aa5399a.jpg)

    - 提供 auth 資訊，可以看到 replica set 只有一個 member

        ![2onemember](https://user-images.githubusercontent.com/3851540/87247556-01ea9000-c487-11ea-9e8f-511ad9c6c09c.jpg)

## 心得

完成程式碼請參考 [yowko/docker-compose-mongodb-replica-set-with-auth-standalone](https://github.com/yowko/docker-compose-mongodb-replica-set-with-auth-standalone)

核心在於 `docker-compose.yaml`，而其中的細部重點是

1. 透過 `enviroment` 設定 mongodb 使用者與密碼
2. mongod 啟動時加上 `--auth`
3. 使用 `healthcheck` 來建立 replica set 與 grant user 權限

雖然筆記內容符合

1. `Replica Set`
2. 有 auth 機制

但就我個人覺得還是差了點，畢竟 MongoDB 官方的 Replica Set 至少也要三個 nodes，所以這就列為改善項目

如果需要透過 docker compose 建立完整三個 nodes 且兼具 auth 功能的 MongoDB Replica Set 可以參考 [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](ttps://blog.yowko.com/docker-compose-mongodb-replica-set-with-auth)

## 參考資訊

1. [Docker Compose 建立 MongoDB Replica Set](https://blog.yowko.com/docker-compose-mongodb-replica-set/)
2. [docker-compose deploy replicaSet in standalone MongoDB cluster and with auth](https://www.cnblogs.com/autohome7390/p/11390465.html)
3. [yowko/docker-compose-mongodb-replica-set-with-auth-standalone](https://github.com/yowko/docker-compose-mongodb-replica-set-with-auth-standalone)
4. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](ttps://blog.yowko.com/docker-compose-mongodb-replica-set-with-auth)
