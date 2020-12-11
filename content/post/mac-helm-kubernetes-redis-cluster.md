---
title: "使用 Helm 在 mac 中的 Kubernetes 上安裝 Redis Cluster"
date: 2020-05-14T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["Kubernetes","Helm","Redis","macOS"]
slug: "mac-helm-kubernetes-redis-cluster"
---

## 使用 Helm 在 Kubernetes 上安裝 Redis Cluster

之前筆記 [在 Kubernetes 部署 Redis Cluster](/kubernetes-redis-cluster/) 曾經紀錄到使用 KubeDB Operator 與 Helm 在 Kubernetes 上安裝 Redis Cluster，最近剛好需要做些 Redis Clsuter 的測試，當然在 vm 上建立是最能完整模擬出測試環境的方式，但每次都重新安裝 Kubernetes 實在有些大廢周章，所以興起在 macOS 上簡單建立環境的想法，順手筆記一下  以免之後要用到又找好一陣子

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.5(43884)

    - Kubernetes v1.15.5

3. Helm v3.2.1
4. Redis image 5.0.7

## 安裝方式

1. 註冊 Helm repository

    ```bash
    helm repo add inspur https://inspur-iop.github.io/charts
    ```

2. 建立 kubernetes namespace

    ```bash
    kubectl create ns redis
    ```

3. 安裝 Redis Cluster

    - 語法

        ```bash
        helm install {helm release name} {helm chart} --namespace {kuberrnetes namespace} --set persistentVolume.storageClass={kubernetes storageClass}
        ```

    - 範例

        > storageClass 不給的話裝不起來，這邊使用 docker for MAC 的 Kubernetes 內建 storageClass - `hostpath`

        ```bash
        helm install redis-cluster inspur/redis-cluster --namespace redis --set persistentVolume.storageClass=hostpath
        ```

        ![2created](https://user-images.githubusercontent.com/3851540/81954996-215a6d80-963c-11ea-93f4-d4f561295009.jpg)

    - 未指定 storageClass 錯誤

        - 錯誤訊息

            ```txt
            Error: failed post-install: timed out waiting for the condition
            ```

        - 錯誤截圖

            ![1errorr](https://user-images.githubusercontent.com/3851540/81954993-20294080-963c-11ea-82c9-dabb00baabe0.jpg)

4. 驗證安裝結果

    - 確認 cluster 是否被正確建立

        ```bash
        redis-cli -c -h redis-cluster-shard0-announce-0.redis.svc.cluster.local -p 6379 cluster info
        ```

        ![3clusterinfo](https://user-images.githubusercontent.com/3851540/81955001-228b9a80-963c-11ea-8fe4-0590bb0f46fd.jpg)

    - cluster 是否有三組 master-clase

        ```bash
        redis-cli -c -h redis-cluster-shard0-announce-0.redis.svc.cluster.local -p 6379 cluster nodes
        ```

        ![4clusternodes](https://user-images.githubusercontent.com/3851540/81955002-23243100-963c-11ea-95ea-2df0eff91dad.jpg)

## 心得

之前在 [在 Kubernetes 部署 Redis Cluster](/kubernetes-redis-cluster/) 時找到最適合的 Helm char 就是 Kubedb 所製作的版本 (其他的 Helm 只建立了 Redis Replication - Master-Slave)，但 Kubedb 在安裝設定上實在相當繁瑣，最近因為升級 Helm 3，重新搜尋這才找到較為便利的版本

另外之前在 Kubernetes 的 storageClass 設定上也讓我相當苦手，剛好嘗試了一下 docker for MAC 的內建功能，可真的大大降低了設定門檻呀

## 參考資訊

1. [在 Kubernetes 部署 Redis Cluster](/kubernetes-redis-cluster/)
2. [redis-cluster](https://hub.helm.sh/charts/inspur/redis-cluster)
