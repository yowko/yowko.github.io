---
title: "MongoDB Exporter 需要的權限"
date: 2021-02-08T21:30:00+08:00
lastmod: 2021-02-08T21:30:31+08:00
draft: false
tags: ["Prometheus","Monitoring","MongoDB"]
slug: "mongodb-exporter-permission"
---

## MongoDB Exporter 需要的權限

之前筆記 [安裝 MongoDB Exporter](/mongodb-exporter) 紀錄到透過 MongoDB Exporter 蒐集 MongoDB metrics 資訊，不過就安全性來看有些不妥：原因是用了 `root` 這個在環境設定(詳細資訊可以參考之前筆記 [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/))中是個 superuser

雖然我們透過 Helm 將 MongoDB Exporter 安裝在 Kubernetes 上，如果不幸 MongoDB Exporter 所使用的帳密外洩，可以想見其他帳密皆外洩的機會很高，但我們不能比爛的，基於最小知識原則還是應該限縮 MongoDB Exporter 存取 MongoDB 的帳號權限

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0(51484)

   - docker engine 20.10.2
   - Kubernetes v1.19.3

3. Helm v3.2.4
4. docker images

    - percona/percona-server-mongodb:4.4.3
    - ssheehy/mongodb-exporter 0.10.0

5. Helm charts

    - prometheus-mongodb-exporter-2.8.1

6. mongodb replicaset

    > 透過 docker compose 來啟動與示範，詳細資訊可以參考之前筆記 [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)

## 權限設定與說明

1. 建立 MongoDB Exporter 專用帳號

    ```js
    db.getSiblingDB("admin").createUser({
        user: "mongodb_exporter",
        pwd: "pass.123",
        roles: [
            { role: "clusterMonitor", db: "admin" },
            { role: "read", db: "local" }
        ]
    })
    ```

2. 權限說明

    - clusterMonitor (官方文件在此 [clusterMonitor](https://docs.mongodb.com/manual/reference/built-in-roles/#clusterMonitor))

        > read-only 的存取權限

    - `role: "read", db: "local"`

        > 因為個別 MongoDB node 的 replicaset sync 狀態會紀錄在 local db 的 `oplog`，MongoDB Exporter 的 replicaset 資訊就會由此取得

## 心得

為什麼需要 `role: "read", db: "local"`，[Percona MongoDB Exporter](https://github.com/percona/mongodb_exporter) 並沒有提到，不知道是不是太基本XD 剛開始看到時還以為是打算，感覺像是需要 `readAnyDatabase` ，查了一些資料才發現原來是 oplog 的關係，筆記一下加深印象

## 參考資訊

1. [Percona MongoDB Exporter](https://github.com/percona/mongodb_exporter)
2. [Prometheus Mongodb Exporter - Correct DB User Permissions](https://stuff.21zoo.com/posts/prometheus-mongodb-exporter-user-permissions/)
3. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)
4. [安裝 MongoDB Exporter](/mongodb-exporter)
5. [clusterMonitor](https://docs.mongodb.com/manual/reference/built-in-roles/#clusterMonitor)
