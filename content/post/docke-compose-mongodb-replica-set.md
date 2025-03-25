---
title: "使用 docker compose 建立 MongoDB replica set"
date: 2025-02-17T00:30:00+08:00
lastmod: 2025-02-17T00:30:31+08:00
draft: false
tags: ["docker","mongodb"]
slug: "docke-compose-mongodb-replica-set"
---

## 使用 docker compose 建立 MongoDB replica set

之前團隊對於 MongoDB 大多還是定位在 document database 上，交易類型的操作還是仰賴傳統 relation database，近期團隊有意減少團隊的 database 種類，嘗試開始逐步使用 MongoDB 來取代 relation database，而我對於 MongoDB 一直停留在 transaction 支援不足的印象，詢問過 chatgpt 之後才發現從 MongoDB 4 開始就支援 transation 了，所以為了能在本機正常開發相關功能，今天就來紀錄一下如何 docker compose 建立 MongoDB replica set

類似需求的主題，之前的筆記也不少：[使用 docker 建立 MongoDB Replica Set](/docker-mongodb-replica-set/)、[Docker Compose 建立 MongoDB Replica Set](/docker-compose-mongodb-replica-set/)、[使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set (Single Node)](/docker-compose-mongodb-replica-set-with-auth-standalone/)、[使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)，但時間都超過四年了，趁著更新 image 版本的機會順便找找看是不是有更適合的做法

## 基本環境設定

1. macOS Sequoia 15.3
2. OrbStack Version 1.10.0 (19021)
3. container image

    - (linux/amd64 - 6b0f3b8898c0) mongo:8.0.4
    - (linux/arm64/v8 - a76cbb4d4f84) mongo:8.0.4

## 設定方式

1. single node
    - 使用 ip

        - 啟動用 entrypoint.sh

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "entrypoint-singlenode-ip.sh">}}

        - docker-compose.yml

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "docker-compose-singlenode-ip.yaml">}}
        - 啟動方式

            `ip={實際使用 ip} docker-compose up -d`

        - 連線字串

            `mongodb://{實際使用 ip}:27017/`

    - 使用 host name (修改 hostfile)

        - 啟動用 entrypoint.sh

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "entrypoint-singlenode-hostfile.sh">}}

        - docker-compose.yml

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "docker-compose-singlenode-hostfile.yaml">}}
        - 修改 hostfile

            `grep -q "127.0.0.1 mongodb" /etc/hosts || echo "127.0.0.1 mongodb" | sudo tee -a /etc/hosts`

        - 連線字串

            `mongodb://mongodb:27017/`

2. cluster (3 nodes)
    - 使用 ip

        - 啟動用 entrypoint.sh

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "entrypoint-cluster-ip.sh">}}

        - docker-compose.yml

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "docker-compose-cluster-ip.yaml">}}

        - 啟動方式

            `ip={實際使用 ip} docker-compose up -d`

        - 連線字串

            `mongodb://{實際使用 ip}:27017,{實際使用 ip}:27018,{實際使用 ip}:27019/`

    - 使用 host name (修改 hostfile)

        - 啟動用 entrypoint.sh

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "entrypoint-cluster-hostfile.sh">}}

        - docker-compose.yml

            {{<gist yowko 38cb91890a34819dc14576f582c36097 "docker-compose-cluster-hostfile.yaml">}}

        - 修改 hostfile

            `grep -q "127.0.0.1 host.docker.internal" /etc/hosts || echo "127.0.0.1 host.docker.internal" | sudo tee -a /etc/hosts`
        - 連線字串

            `mongodb://host.docker.internal:27017,host.docker.internal:27018,host.docker.internal:27019/`

## 心得

在進行實際功能測試時，持續收到 authentication error 的訊息，從連線字串的格式、container 環境變數的設定方式、docker compose yaml 的跳脫字元、最後發現 arm64v8 的 image 無法套用 `MONGO_INITDB_ROOT_USERNAME` 與 `MONGO_INITDB_ROOT_PASSWORD` ([GitHub issuse：MongoDB container environment variables not applied for authentication](https://github.com/docker-library/mongo/issues/715) windows 版似乎有相同問題)，換成 amd64 的 image 可以正常運作

完整內容請參考：[GitHub:yowko/docke-compose-mongodb-replica-set](https://github.com/yowko/docke-compose-mongodb-replica-set)

## 參考資訊

1. [使用 docker 建立 MongoDB Replica Set](/docker-mongodb-replica-set/)
2. [Docker Compose 建立 MongoDB Replica Set](/docker-compose-mongodb-replica-set/)
3. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set (Single Node)](/docker-compose-mongodb-replica-set-with-auth-standalone/)
4. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)
5. [GitHub:yowko/docke-compose-mongodb-replica-set](https://github.com/yowko/docke-compose-mongodb-replica-set)
