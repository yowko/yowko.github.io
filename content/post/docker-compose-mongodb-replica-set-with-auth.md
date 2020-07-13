---
title: "使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set"
date: 2020-07-12T21:30:00+08:00
lastmod: 2020-07-13T09:30:31+08:00
draft: false
tags: ["MongoDB","Docker"]
slug: "docker-compose-mongodb-replica-set-with-auth"
---

## 使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set

之前在筆記 [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set (Single Node)](https://blog.yowko.com/docker-compose-mongodb-replica-set-with-auth-standalone/) 紀錄到如何使用 docker compose 建立啟用 auth 機制的單一 node MongoDB Replica Set，但單一 node 的 MongoDB Replica Set 對我來說不太滿足 MongoDB Replica Set 的定義，所以嘗試了三個 nodes ( primary + secondary + secondary) 的做法

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker images

    - mongo:4.2.8-bionic

## 建立方式

1. 建立 keyfile

    ```bash
    openssl rand -base64 756 > keyfile
    chmod 400 keyfile
    ```

2. docker-compose.yaml

    ```yaml
    version: "3.4"

    services:
      mongotmp:
        container_name: mongotmp
        image: mongo:4.2.8-bionic
        volumes:
          - ./keyfile:/keyfile
        environment:
          - MONGO_INITDB_ROOT_USERNAME=${username}
          - MONGO_INITDB_ROOT_PASSWORD=${password}
        ports:
          - 27047:27047
        healthcheck:
          test:
            [
              "CMD",
              "mongo",
              "admin",
              "--port",
              "27047",
              "-u",
              "${username}",
              "-p",
              "${password}",
              "--eval",
              'rs.initiate( { _id : "rs0",members: [{ _id: 0,host: "mongotmp:27047" }]});db.getSiblingDB("admin").createUser({user: "${username}",pwd: "${password}",roles: [{role: "userAdminAnyDatabase",db:"admin"},{role: "clusterAdmin",db:"admin"}]})',
            ]
          interval: 10s
          start_period: 10s
        command: [
            "--transitionToAuth",
            "-keyFile",
            "/keyfile",
            "--replSet",
            "rs0",
            "--bind_ip_all",
            "--port",
            "27047"
          ]
      mongo2:
        container_name: mongo2
        image: mongo:4.2.8-bionic
        volumes:
          - ./keyfile:/keyfile
        ports:
          - 27027:27027
        depends_on:
          - mongotmp
        healthcheck:
          test:
            [
              "CMD",
              "mongo",
              "--host",
              "mongotmp",
              "--port",
              "27047",
              "admin",
              "-u",
              "${username}",
              "-p",
              "${password}",
              "--eval",
              'rs.add( { host: "mongo2:27027"} )',
            ]
          interval: 10s
          start_period: 20s
        command:
          [
            "-keyFile",
            "/keyfile",
            "--replSet",
            "rs0",
            "--bind_ip_all",
            "--port",
            "27027",
          ]
      mongo3:
        container_name: mongo3
        image: mongo:4.2.8-bionic
        volumes:
          - ./keyfile:/keyfile
        ports:
          - 27037:27037
        depends_on:
          - mongotmp
          - mongo2
        healthcheck:
          test:
            [
              "CMD",
              "mongo",
              "--host",
              "mongotmp",
              "--port",
              "27047",
              "admin",
              "-u",
              "${username}",
              "-p",
              "${password}",
              "--eval",
              'rs.add( { host: "mongo3:27037"} )',
            ]
          interval: 10s
          start_period: 30s
        command:
          [
            "-keyFile",
            "/keyfile",
            "--replSet",
            "rs0",
            "--bind_ip_all",
            "--port",
            "27037",
          ]
      mongo1:
        container_name: mongo1
        image: mongo:4.2.8-bionic
        volumes:
          - ./keyfile:/keyfile
        ports:
          - 27017:27017
        depends_on:
          - mongotmp
          - mongo2
          - mongo3
        healthcheck:
          test:
            [
              "CMD",
              "mongo",
              "--host",
              "mongotmp",
              "--port",
              "27047",
              "admin",
              "-u",
              "${username}",
              "-p",
              "${password}",
              "--eval",
              'rs.add( { host: "mongo1:27017"} );db.adminCommand( { replSetStepDown: 1,force:true } );',
            ]
          interval: 10s
          start_period: 40s
        command:
          [
            "-keyFile",
            "/keyfile",
            "--replSet",
            "rs0",
            "--bind_ip_all"
          ]
      remover:
        container_name: remover
        image: mongo:4.2.8-bionic
        volumes:
          - ./keyfile:/keyfile
        depends_on:
          - mongo1
        healthcheck:
          test: /usr/bin/mongo "mongodb://${username}:${password}@mongo1:27017,mongo2:27027,mongo3:27037/admin?replicaSet=rs0" --eval 'rs.remove("mongotmp:27047")'
          interval: 10s
          start_period: 45s
        command:
          [
            "-keyFile",
            "/keyfile",
            "--replSet",
            "rs0",
            "--bind_ip_all"
          ]
    ```

3. 直接使用 docker compose 啟動

    > 如果想要移除中繼 container ，請直接跳至 `步驟 4` 或是執行此步驟後手動移除

    - 語法

        ```bash
        username={username} password={password} docker-compose up -d
        ```

    - 範例

        ```bash
        username=root password=pass.123 docker-compose up -d
        ```

4. (optional) 將移除非必要 container 包裝成 shell

    > 把中介性質的 container 停掉，不在意資源使用狀況的話，這個步驟可以直接略過

    - start.sh

        > 請記得將 username 與 password 換成需要的值

        - 語法

            ```bash
            username={username} password={password} docker-compose up -d
            sleep 50
            docker-compose stop mongotmp remover
            ```

        - 範例

            ```bash
            username=root password=pass.123 docker-compose up -d
            sleep 50
            docker-compose stop mongotmp remover
            ```

    - 啟動 mongodb replica set

        ```bash
        sh ./start.sh
        ```

5. 實際使用

    - 沒有提供 auth，無法取得 replica set 資訊

        ![1noauth](https://user-images.githubusercontent.com/3851540/87247548-ff883600-c486-11ea-85a4-72419aa5399a.jpg)

    - 提供 auth 資訊，可以看到 replica set 已有三個 member

        ![2members](https://user-images.githubusercontent.com/3851540/87248991-129f0400-c48f-11ea-8e92-84a96cc08d3e.jpg)

## 心得

完成程式碼請參考 [yowko/docker-compose-mongodb-replica-set-with-auth](https://github.com/yowko/docker-compose-mongodb-replica-set-with-auth)

雖然 mongo 用了好多年了，但每次遇到 mongo 不論是什麼角色，mongo 還是常常讓我覺得棘手，這次也一樣：反覆看著官方文件，還是不知道問題出在哪兒、該如何解決XD

以下簡單說明這份 docker-compose 的意圖

1. mongotmp

    - 有基本的 user、password
    - 建立 replica set
    - 建立 admin 與 clusteradmin user 用
    - 啟動時使用 `keyfile` 做為後續 member 驗證
    - `transitionToAuth` 讓後續 member 可以不驗證，但這個參數只要存在就會讓 mongo 可以不驗證

2. mongo 2, mongo3

    - 在 healthcheck 時連線至 mongotmp 加入 replica set

3. mongo 1

    - 在 healthcheck 時連線至 mongotmp 加入 replica set 並將 mongotmp 降級 (primary --> secondary)

4. remover

    - 將 mongotmp 自 replica set 中移除 (讓所有 member 再沒有 `transitionToAuth` 以完全啟用 auth)

5. 最後將完成階段性任務的 `mongotmp` 與 `remover` 兩個 docker-compose service 移除

說實話個人覺得寫得不是很漂亮，但這是目前可以完成目標的最佳解法，暫時先這樣，待日後想到其他方式再來更新囉

## 參考資訊

1. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set (Single Node)](https://blog.yowko.com/docker-compose-mongodb-replica-set-with-auth-standalone/)
2. [yowko/docker-compose-mongodb-replica-set-with-auth](https://github.com/yowko/docker-compose-mongodb-replica-set-with-auth)
3. [Update Replica Set to Keyfile Authentication](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/)
4. [Update Replica Set to Keyfile Authentication (No Downtime)](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set-without-downtime/)
