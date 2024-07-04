---
title: "使用 Helm 安裝 Redis Cluster"
date: 2024-06-06T00:30:00+08:00
lastmod: 2024-06-06T00:30:31+08:00
draft: false
tags: ["helm","redis"]
slug: "helm-redis-cluster"
---

## 使用 Helm 安裝 Redis Cluster

之前筆記 [在 Kubernetes 部署 Redis Cluster](/kubernetes-redis-cluster/) 與 [使用 Helm 在 Kubernetes 上安裝 Redis Cluster](/mac-helm-kubernetes-redis-cluster/) 紀錄到如何使用 Helm 在 Kubernetes 上安裝 Redis Cluster，最近因為專案需要打算首次將 Redis 安裝在 Kubernetes 上供正式服務使用，經過了四、五年的演進，以現在的觀點來看，當時的方式已經不符現在的標準，所以趁著這個機會更新一下筆記

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack Version 1.6.1 (17010)
    - Kubernetes Client : v1.29.4
    - Kubernetes Server : v1.29.3+orb1
- Helm Version v3.14.3
- Helm Chart
    - bitnamicharts/redis-cluster 10.2.2
- docker image

    - redis:7.2.5-debian-12-r0

## 安裝步驟

- 預設安裝

    > - 使用 oci 來安裝 helm chart，需要 helm 3.8 以上版本
    > - 未指定 redis 密碼，預設會自動產生一組密碼 (可透過 `kubectl get secret redis-cluster -o jsonpath="{.data.redis-password}" | base64 --decode` 取得)

    ```bash
    helm install redis oci://registry-1.docker.io/bitnamicharts/redis-cluster
    ```

- 設定密碼

    > 將密碼設定為 `pass.123`

    ```bash
    helm install redis --set "password=pass.123" oci://registry-1.docker.io/bitnamicharts/redis-cluster
    ```

## 心得

1. 透過 statefulset 建立 6 個 redis instance pod

    ![1statefulset](https://github.com/yowko/picsbed/assets/3851540/4cb1bc2a-8397-402a-bf7b-42d48c40660f)

    ![2pods](https://github.com/yowko/picsbed/assets/3851540/c71b580b-1fde-438f-96b5-60541af07753)

2. 單一 service 做為 redis cluster 的入口

    > 在相同 Kubernetes namespace 中的 application 連線使用 `{helm release name}-redis-cluster:6379`

    ![3redissvc](https://github.com/yowko/picsbed/assets/3851540/a6ce00d0-6806-465c-af21-40b3fba7b126)

3. 透過 headless service 來提供 redis cluster 的服務

    ![4headless](https://github.com/yowko/picsbed/assets/3851540/211630f5-fec6-465a-8d8d-a0c12ff81d57)

## 參考資訊

1. [Bitnami package for Redis(R) Cluster](https://artifacthub.io/packages/helm/bitnami/redis-cluster)
2. [GitHub:bitnami/redis-cluster](https://github.com/bitnami/charts/tree/main/bitnami/redis-cluster)
3. [在 Kubernetes 部署 Redis Cluster](/kubernetes-redis-cluster/)
4. [使用 Helm 在 Kubernetes 上安裝 Redis Cluster](/mac-helm-kubernetes-redis-cluster/)
5. [Use OCI-based registries](https://helm.sh/docs/topics/registries/)
