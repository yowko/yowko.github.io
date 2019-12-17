---
title: "使用客製 config 在 kubenetes 上建立 Redis cluster"
date: 2019-12-17T21:30:00+08:00
lastmod: 2019-12-17T21:30:31+08:00
draft: false
tags: ["Kubernetes","redis"]
slug: "redis-config-cluster-kubernetes"
---

## 使用客製 config 在 kubenetes 上建立 Redis cluster

之前筆記 [在 Kubernetes 部署 Redis Cluster](https://blog.yowko.com/kubernetes-redis-cluster/) 紀錄到如何使用 KubeDB Operator 來建立 redis cluster，不過之前筆記內容是透過預設值來進行安裝與設定，而在面對不同情境的現實需求(eg.改變最大連線數、指定 log 位置、RDB 與 AOF 的策略....etc)下就顯得有些捉襟見肘，所以今天就來紀錄一下如何在 Kubernetes 上透過 KubeDB 安裝 redis cluster 時使用自有設定值

## 基本環境說明

> 以下 VM 透過 Azure 建立

1. CentOS 7.6 * 4

    - Standard B2ms
    - 2 vcpus, 8 GiB memory
    - 200 GB Standard SSD

2. VM 配置說明

    > 請確保各個 node 間可以透過 host name 連線

    Name|IP|用途
    :---|:---|:---
    node1| 10.0.0.4|ansible-client
    node2| 10.0.0.5|master,etcd
    node3| 10.0.0.6|node,etcd
    node4| 10.0.0.7|node,etcd

3. Kubernetes 1.16.3
4. Kubespray 2.12
5. <span style="color:red">Helm v2.14.3</spna>

    > 測試時如高於這個版本，會造成安裝失敗

6. <span style="color:red">KubeDB v0.13.0-rc.0</span>

    > 測試時如低於這個版本，會造成安裝失敗

7. Redis 5.0.3

## 安裝 KubeDB Operator

> 以下使用 Helm 安裝，也可以使用 script 來執行安裝，請參考 [Installation Guide](https://kubedb.com/docs/v0.13.0-rc.0/setup/install/) 的 `Script` tab 內容

1. 加入 appscode (KudeDB 的開發公司) 的 helm repo

    ```bash
    helm repo add appscode https://charts.appscode.com/stable/
    ```

2. 更新 Helm repo

    ```bash
    helm repo update
    ```

3. 安裝 kubedb operator chart

    ```bash
    helm install appscode/kubedb --name kubedb-operator --version v0.13.0-rc.0 --set apiserver.enableMutatingWebhook=false --set apiserver.enableValidatingWebhook=false --namespace kube-system
    ```

    - 未關閉 `apiserver.enableMutatingWebhook` 錯誤

        > 使用 kubedb 建立 redis 時會出現錯誤

        - 錯誤訊息

            ```txt
            Error from server (InternalError): error when creating "redis-cluster.yaml": Internal error occurred: failed calling webhook "redis.mutators.kubedb.com": the server is currently unable to handle the request
            ```

        - 錯誤截圖

            ![4webhookfail](https://user-images.githubusercontent.com/3851540/63025853-a9102180-bedc-11e9-883c-20241fc7c5d0.png)

    - 未關閉 `apiserver.enableValidatingWebhook` 錯誤

        > 使用 kubedb 建立 redis 時會出現錯誤

        - 錯誤訊息

            ```txt
            Error from server (InternalError): error when creating "redis-cluster.yaml": Internal error occurred: failed calling webhook "redis.validators.kubedb.com": the server is currently unable to handle the request
            ```

        - 錯誤截圖

            ![5validatefail](https://user-images.githubusercontent.com/3851540/63025854-a9102180-bedc-11e9-8200-95ac49c21370.png)

4. 確認成功註冊 CRD

    ```bash
    kubectl get crds -l app=kubedb
    ```

    ![1registcrd](https://user-images.githubusercontent.com/3851540/63025850-a8778b00-bedc-11e9-9af2-5582ac74f2c2.png)

5. 安裝 KubeDB catalog

    ```bash
    helm install appscode/kubedb-catalog --name kubedb-catalog --version v0.13.0-rc.0   --namespace kube-system
    ```

6. 確認成功安裝

    > KubeDB operator pods 是 `Running`

    ```bash
    kubectl get pods --all-namespaces -l app=kubedb
    ```

    ![2operatorrunning](https://user-images.githubusercontent.com/3851540/63027651-e88c3d00-bedf-11e9-80c4-faf05543bc4a.png)

## 安裝 KubeDB CLI

1. 下載編譯完成的 sh

    ```bash
    wget -O kubedb https://github.com/kubedb/cli/releases/download/v0.13.0-rc.0/kubedb-linux-amd64 \
    && chmod +x kubedb \
    && sudo mv kubedb /usr/local/bin/
    ```

2. 確認成功安裝

    ```bash
    kubedb version
    ```

    ![1kubedbversion](https://user-images.githubusercontent.com/3851540/70963618-fe934f80-20c3-11ea-8223-b51abbf19fc3.png)

## 準備 redis.conf 進行安裝

1. 建立測試用 namespace

    ```bash
    kubectl create ns demo
    ```

2. redis.conf

    ```txt
    cluster-enabled yes
    cluster-config-file /data/nodes.conf
    cluster-node-timeout 5000
    cluster-migration-barrier 1
    dir /data
    appendonly yes

    maxclients 50000

    requirepass  pass.123
    masterauth pass.123
    rename-command FLUSHALL ""
    rename-command FLUSHDB ""
    ```

3. 將 redis.conf 建立成一個 configMap

    ```bash
    kubectl create configmap -n demo rd-custom-config --from-file=./redis.conf
    ```

4. redis cluster 安裝用 yaml

    > 使用上面加入的 configMap 來安裝 (透過 `spec.configSource.configMap.name`)

    ```yaml
    apiVersion: kubedb.com/v1alpha1
    kind: Redis
    metadata:
      name: redis-cluster
      namespace: demo
    spec:
      version: 5.0.3-v1
      configSource:
        configMap:
          name: rd-custom-config
      mode: Cluster
      cluster:
        master: 3
        replicas: 1
      storageType: Durable
      storage:
        resources:
          requests:
            storage: 1Gi
        storageClassName: "local-redis"
        accessModes:
        - ReadWriteOnce
      terminationPolicy: Pause
      updateStrategy:
        type: RollingUpdate
    ```

5. 使用 `redis-cluster.yaml` 建立 Redis

    ```bash
    kubedb create -f redis-cluster.yaml
    ```

6. 確認成功安裝

    - 檢查 type 為 `redis` 且名稱為 `redis-cluster` 的所有 resource 狀態

        ```bash
        kubedb describe redis -n demo redis-cluster
        ```

    - 成功建立 三 個 StatefulSet 且各有 二 個 pod 運行中

        ```bash
        kubedb get statefulset -n demo
        ```

        ![8statefulsetok](https://user-images.githubusercontent.com/3851540/63025859-a9a8b800-bedc-11e9-8da8-4cce376fca18.png)

    - 連線至 redis 中確認 cluster 狀態

        ```bash
        kubectl -n demo exec -it redis-cluster-shard0-0 -- redis-cli -a pass.123 cluster nodes
        ```

        ![2clusterinfo](https://user-images.githubusercontent.com/3851540/70963619-ff2be600-20c3-11ea-943c-1d73a1c0c072.png)

## 心得

KubeDB 在 redis 安裝時使用客製 config，我實測下 `v0.12.0` 會無法成功建立 cluster 需改用 `KubeDB v0.13.0-rc.0`，官方 GitHub 有其他人反應相同 issue：[[redis] Redis cluster with custom config file not working](https://github.com/kubedb/project/issues/490)，另個問題是 Helm v2.16.3 也無法正確完成安裝，需要改用 `Helm v2.14.3` 就可以暫時解決問題，官方 GitHub 有其他人反應相同 issue：[[Helm Chart catalog fails to be installed - validators.kubedb.com/v1alpha1: the server is currently unable to handle the request](https://github.com/kubedb/project/issues/671)

其中 [[redis] Redis cluster with custom config file not working](https://github.com/kubedb/project/issues/490) 官方人員提到因為 securitty 因素，不支援 `requirepass` 設定，實測下倒是 ok

不知道是太少人用還是穩定性真的待加強，我自己照著官方上的文件操作數十次，從來不曾一次搞定過，總是得要一直翻查 issue 看看有沒有人遇到一樣問題跟如何解決，這點在實際使用上個人覺得頗為困擾，但畢竟人家也是佛心免費提供，實在不忍苛責呀

## 參考資訊

1. [在 Kubernetes 部署 Redis Cluster](https://blog.yowko.com/kubernetes-redis-cluster/)
2. [Using Custom Configuration File](https://kubedb.com/docs/0.12.0/guides/redis/custom-config/using-custom-config/)
3. [[redis] Redis cluster with custom config file not working](https://github.com/kubedb/project/issues/490)
4. [Installation Guide](https://kubedb.com/docs/v0.13.0-rc.0/setup/install/)
5. [Helm Chart catalog fails to be installed - validators.kubedb.com/v1alpha1: the server is currently unable to handle the request](https://github.com/kubedb/project/issues/671)
