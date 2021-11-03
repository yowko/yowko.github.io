---
title: "安裝 MongoDB Exporter"
date: 2021-02-07T21:30:00+08:00
lastmod: 2021-02-07T21:30:31+08:00
draft: false
tags: ["Prometheus","Monitoring","MongoDB"]
slug: "mongodb-exporter"
---

## 安裝 MongoDB Exporter

之前筆記 [Prometheus 搭配 Alertmanager 發送警示](/prometheus-alertmanager/) 紀錄到如何使用 Prometheus 與 Alertmanager 來進行監控並主動發生通知，而 MongoDB 也可以是 Prometheus 監控的對象，透過安裝 MongoDB Exporter 將 metrics 資訊透過 http server 方式揭露再透過 Prometheus 主動 pull 即可將 MongoDB 相關資訊儲存至 Prometheus 中

今天就來快速紀錄一下 如何使用 Helm 來安裝 MongoDB Exporter

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

## 安裝方式

1. 設定 Helm repo

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm update
    ```

2. 安裝 exporter

    - 語法

        ```bash
        helm upgrade --install mongoexportter prometheus-community/prometheus-mongodb-exporter --set mongodb.uri="{username}:{passowrd}@{mongo_host1}\:{mongo_port1}\,{mongo_host2}\:{mongo_port2}\,{mongo_host3}\:{mongo_port3}"
        ```

    - 範例

        > 這邊以 host ip 為例，helm value 遇到 `.` `[` `,` `=` 需要加上跳脫字元 `\`

        ```bash
        helm upgrade --install mongoexportter prometheus-community/prometheus-mongodb-exporter --set mongodb.uri="root:pass\.123@192\.168\.50\.97:27017\,192\.168\.50\.97:27027\,192\.168\.50\.97:27037"
        ```

3. 實際效果

    - 將 mongodb export 做 port forward 以利 debug

        ```bash
        kubectl port-forward service/mongoexportter-prometheus-mongodb-exporter 9216
        ```

    - access mongodb export 所取得的 metrics 資訊

        ```bash
        curl http://127.0.0.1:9216/metrics
        ```

        ![1metrics](https://user-images.githubusercontent.com/3851540/107150865-e2730180-699a-11eb-98a0-f0bc8b7f7649.png)

## 心得

helm value 的 `mongodb.uri` 我安裝時如果加上 `mongodb://` 的前綴就會出現 auth fail 的狀況，看 log 是沒吃到 user 與 password，另外就是 `port` 我只要明確指定就會出錯，這兩個問題不知道是不是因為 set value 才造成的，但過去我應該是造描述中的用法來安裝的，這就是過太久才筆記的問題XD

## 參考資訊

1. [Percona MongoDB Exporter](https://github.com/percona/mongodb_exporter)
2. [prometheus-community/helm-charts/charts/prometheus-mongodb-exporter](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-mongodb-exporter)
3. [Helm: How to Override Value with Periods in Name](https://stackoverflow.com/a/66031634)
4. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)
