---
title: "GKE 透過 Private IP 連線 Cloud SQL"
date: 2020-12-06T21:30:00+08:00
lastmod: 2020-12-06T21:30:31+08:00
draft: false
tags: ["GCP","Kubernetes"]
slug: "gke-private-sql-cloud-sql"
---

## GKE 透過 Private IP 連線 Cloud SQL

因為團隊中沒有傳統 DBA (只有 BA 兼任 DBA 哈哈)，所以在 db 的選擇上一直都偏好使用外部服務 (自建環境時與其他團隊共享 DBA 資源) 所以在評估 GCP 時也就選擇使用 Cloud SQL，當然使用 SaaS 服務的限制： 1. 沒辦法進行細微設定  2. 底層 patch 可控程度低 3. 語法受限 4. 硬體效能受限 5. 價格昂貴 .... 等等，都已經被納入評估，最後決定因素主要還是在 `便利` 與 `彈性`，GCP 管理大多數情況下應該會優於外行人管理吧 XD

我 google 了一下，網路上的文章跟官方建議都使用 Cloud SQL Proxy 為主，雖然我認同 Cloud SQLProxy 的安全性，但我覺得部署與設定的動作太多了，對測試環境還是 on-premiss 機房的我們而已，要兼容修改的範圍過大，所以才想從網路這層著手

## 基本環境說明

1. GKE (1.17.13-gke.2001)
2. Cloud SQL (MySQL 8.0)

## 設定方式

1. 建立 GKE (已有 GKE instance 可忽略)

    - 建立叢集

        ![1create-gke](https://user-images.githubusercontent.com/3851540/101283015-2cacfa80-3813-11eb-94cf-a13a3743d026.png)

    - 設定 Kubernetes networking

        > 使用 `default` VPC

        ![2networking](https://user-images.githubusercontent.com/3851540/101283020-30d91800-3813-11eb-8bb2-25d51b5bac5d.png)

2. 建立 Cloud SQL

    - 建立執行個體

        ![3createcloudsql](https://user-images.githubusercontent.com/3851540/101283021-3171ae80-3813-11eb-9882-693de8d40c96.png)

    - 選擇資料庫引擎

        > 這邊以 `mysql` 為例

        ![4dbengine](https://user-images.githubusercontent.com/3851540/101283023-320a4500-3813-11eb-88f8-f3b512606b3d.png)

    - 設定 db networking

        > 使用 `Private IP` 並指定 `default` VPC

        ![5privateip](https://user-images.githubusercontent.com/3851540/101283024-32a2db80-3813-11eb-843f-3b59a47f258f.png)

## 驗證方式

1. 部署 container

    > 以下使用之前筆記 [使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期](https://blog.yowko.com/kubernetes-liveness-aspdotnet-core-grpc/) 的 image 來進行示範

    ```bash
    kubectl create deployment test  --image=gcr.io/thermal-setup-295904/healthcheck:healthy
    ```

2. 至 container 直接連線

    - 語法

        ```bash
        kubectl exec -it {pod name} -- bash
        ```

    - 範例

        ```bash
        kubectl exec -it test-c57fc89dc-jf5dx -- bash
        ```

3. 安裝 mysql client

    > 請依 base image os 自行調整

    ```bash
    apt update && apt install -y default-mysql-client
    ```

4. 使用 Private IP 連線至 Cloud SQL

    - 取得 Cloud SQL Private IP

        ![6getprivateip](https://user-images.githubusercontent.com/3851540/101283026-333b7200-3813-11eb-9d66-0d42ba0fce94.png)

    - 語法

        ```bash
        mysql -h {cloud sql private ip} -u{username} -p{passowrd}
        ```

    - 範例

        ```bash
        mysql -h 10.65.195.3 -uroot -ppass.123
        ```

    - 實際結果

        ![7result](https://user-images.githubusercontent.com/3851540/101283027-333b7200-3813-11eb-85f2-7e002e3d5b12.png)

## 心得

經過一連串測試調整一直遇到連線不通的問題，後來重新看了說明文件才發現漏看了一段重點 @@"

![8samevpc](https://user-images.githubusercontent.com/3851540/101283028-33d40880-3813-11eb-8cf7-2defe4ee4190.png)

這充份說明：別自以為是，不管做什麼還是需要閱讀公開說明書XD

## 參考資訊

1. [Connecting from Google Kubernetes Engine](https://cloud.google.com/sql/docs/mysql/connect-kubernetes-engine)
2. [使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期](https://blog.yowko.com/kubernetes-liveness-aspdotnet-core-grpc/)
