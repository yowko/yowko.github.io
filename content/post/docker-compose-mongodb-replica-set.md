---
title: "Docker Compose 建立 MongoDB Replica Set"
date: 2020-05-10T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["MongoDB"]
slug: "docker-compose-mongodb-replica-set"
---

## Docker Compose 建立 MongoDB Replica Set

因為之前筆記 [MongoDB Cli Replica Set 連線方式](/mongodb-cli-replica-set/) 需要建立測試環境，想起最早之前的筆記 [使用 docker 建立 MongoDB Replica Set](/docker-mongodb-replica-set/) 但使用起來並不是很好用，所以興起筆記其他建立 MongoDB Replica Set 方式的念頭

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.5(43884)
3. MongoDB 4.2.6

## 使用方式

1. docker-compose.yml

    - 將三個 mongo instance 完整分開，以利於 host 機器的 port mapping

        > mongod 啟動指令中設定 `--port` 即可

    - 透過 docker-compose 的 `healthcheck` 功能來設定 MongoDB Replica Set

    ```yml
    version: "3.7"
    services:
      mongo1:
        container_name: mongo1
        image: mongo
        ports:
          - 27017:27017
        restart: always
        entrypoint: [ "mongod","--port","27017", "--bind_ip_all",   "--replSet", "rs0" ]
      mongo2:
        container_name: mongo2
        image: mongo
        ports:
          - 27027:27027
        restart: always
        entrypoint: [ "mongod","--port","27027", "--bind_ip_all",   "--replSet", "rs0" ]
      mongo3:
        container_name: mongo3
        image: mongo
        ports:
          - 27037:27037
        restart: always
        entrypoint: [ "mongod","--port","27037", "--bind_ip_all",   "--replSet", "rs0" ]
        healthcheck:
          test: ["CMD","mongo","--host","mongo1","--port","27017",  "--eval", 'rs.initiate( { _id : "rs0",members: [{ _id: 0,     host: "mongo1:27017" },{ _id: 1, host: "mongo2:27027" },{   _id: 2, host: "mongo3:27037" }   ]})']
          interval: 15s
          timeout: 10s
          retries: 3
          start_period: 10s
    ```

2. 設定 hosts

    > mongo 內部只認得設定 Replica Set 的 member host，但 host 機器不認得 container name

    ```bash
    sudo echo "127.0.0.1 mongo1\n127.0.0.1 mongo2\n127.0.0.1 mongo3" >> /etc/hosts
    ```

3. 啟動 MongoDB Replica Set

    > 執行指令後需要等個幾十秒才會設定完成

    ```bash
    docker-compose up -d
    ```

## 心得

跟之前 [使用 docker 建立 MongoDB Replica Set](/docker-mongodb-replica-set/) 相比，少了一個 `creator` 的 dockerfile 需要維護 (之前多一個 `creator` 是因為 mongo 啟動當下無法立即接受連線，而我嘗試 sleep 也失敗，不得已才多一個 container 來暫解)，整體說來更為簡潔，也因為將各個 instance 完整切開，所以從外部的連線也可以正常運作

source code 請參考 [yowko/docker-compose-create-mongodb-replica-set](https://github.com/yowko/docker-compose-create-mongodb-replica-set)

## 參考資訊

1. [MongoDB Cli Replica Set 連線方式](/mongodb-cli-replica-set/)
2. [使用 docker 建立 MongoDB Replica Set](/docker-mongodb-replica-set/)
3. [mongod](https://docs.mongodb.com/manual/reference/program/mongod/)
4. [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/#healthcheck )
5. [yowko/docker-compose-create-mongodb-replica-set](https://github.com/yowko/docker-compose-create-mongodb-replica-set)
