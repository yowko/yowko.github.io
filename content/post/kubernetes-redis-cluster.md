---
title: "在 Kubernetes 部署 Redis Cluster"
date: 2019-08-15T01:30:00+08:00
lastmod: 2019-08-15T01:30:31+08:00
draft: false
tags: ["Kubernetes","Redis"]
slug: "kubernetes-redis-cluster"
---

## 在 Kubernetes 部署 Redis Cluster

最近在 Kubernetes 上部暑相關基礎服務，大部份的服務都有對應的 Helm 可以使用，沒有遇到太多問題，唯獨 Redis，因為團隊使用 Redis Cluster,，而 Helm 官方的兩個 chart [helm/charts/stable/redis](https://github.com/helm/charts/tree/master/stable/redis) 與 [helm/charts/stable/redis-ha](https://github.com/helm/charts/tree/master/stable/redis-ha) 都無法滿足需求，雖然 [helm/charts/stable/redis](https://github.com/helm/charts/tree/master/stable/redis) 具有 cluster 相關設定，但實測下只是 master-slave + sentinel 的 cluster，並非 Redis cluster，後來同事建議試試 KubeDB 的 Redis cluster，過程中遇到不少問題，紀錄一下避免改天部署時還不起來該怎麼解決問題

## 基本環境說明

> 以下 VM 透過 Azure 建立

1. CentOS 7.6 * 3

    - Standard B2ms
    - 2 vcpus, 8 GiB memory
    - 200 GB Standard SSD

2. VM 配置說明

    > 請確保各個 node 間可以透過 host name 連線

    Name|IP|用途
    :---|:---|:---
    node1| 10.0.0.4|ansible-client,master,etcd
    node2| 10.0.0.5|node,etcd
    node3| 10.0.0.6|node,etcd

3. Kubernetes 1.14.3
4. Kubespray 2.10
5. Helm 2.13.1
6. KubeDB 0.12.0
7. Redis 5.0.3

## 安裝 KubeDB Operator

> 以下使用 Helm 安裝，也可以使用 script 來執行安裝，請參考 [Installation Guide](https://kubedb.com/docs/0.12.0/setup/install/) 的 `Script` tab 內容

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
    helm install appscode/kubedb --name kubedb-operator --version 0.12.0 --set apiserver.enableMutatingWebhook=false --set apiserver.enableValidatingWebhook=false --namespace kube-system
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
    helm install appscode/kubedb-catalog --name kubedb-catalog --version 0.12.0 --namespace kube-system
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
    wget -O kubedb https://github.com/kubedb/cli/releases/download/0.12.0/kubedb-linux-amd64 \
    && chmod +x kubedb \
    && sudo mv kubedb /usr/local/bin/
    ```

2. 確認成功安裝

    ```bash
    kubedb version
    ```

    ![3kubedbversion](https://user-images.githubusercontent.com/3851540/63025852-a9102180-bedc-11e9-91f4-1386d0b5d49f.png)

## 準備 StorageClass 與 Persistent Volume

1. 建立 StorageClass

    - 新增 `strageclass.yaml`

        ```yaml
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: local-redis
        provisioner: kubernetes.io/no-provisioner
        volumeBindingMode: WaitForFirstConsumer
        ```

    - 使用 `strageclass.yaml` 建立 StorageClass

        ```bash
        kubectl apply -f strageclass.yaml
        ```

    - 未建立錯誤 StorageClass

        > 使用 `kubedb describe rd -n demo redis-cluster` 可以看到錯誤

        - 錯誤訊息

            ```txt
            spec.storage.storageClassName "local-redis" not found
            ```

        - 錯誤截圖

            ![6storageclasserror](https://user-images.githubusercontent.com/3851540/63025857-a9a8b800-bedc-11e9-9f34-bee823ca8a64.png)

2. 建立 Persistent Volume

    > 我個人開發測試使用 Local Persistent Volume， redis cluster 有 6 個 node，每個 node 都需要一個 volume

    - 建立六個資料夾並給予權限

        > 以下動作在 `node1` 上執行 (會影響 pv.yaml 設定)，請自行替換 {vol1} 直至建立 六 個資料夾

        ```bash
        mkdir -p /mnt/disk/vol1
        chcon -Rt svirt_sandbox_file_t /mnt/disk/vol1
        chmod 777 /mnt/disk/vol1
        ```

    - 新增六個 pv.yaml

        > 請自行替換 `metadata.name`,`spec.local.path` 直至建立 六 個 pv.yaml，另外如果資料夾不是建立在 `node1` 上的請修改 `spec.nodeAffinity.required.nodeSelectorTerms.matchExpressions.key.values`

            ```yaml
            apiVersion: v1
            kind: PersistentVolume
            metadata:
            name: local-pv1
            spec:
            capacity:
                storage: 1Gi
            volumeMode: Filesystem
            accessModes:
            - ReadWriteOnce
            persistentVolumeReclaimPolicy: Delete
            storageClassName: local-redis
            local:
                path: /mnt/disk/vol1
            nodeAffinity:
                required:
                nodeSelectorTerms:
                - matchExpressions:
                    - key: kubernetes.io/hostname
                    operator: In
                    values:
                    - node1
            ```


    - 使用 pv.yaml 建立 Local Persistent Volume

        > 重複 六 次直至建立 六 個 pv

        ```bash
        kubectl apply -f pv1.yaml
        ```

    - 未建立 PV 錯誤

        > 使用 `kubectl -n demo describe pod redis-cluster-shard0-0` 可以看見錯誤訊息

        - 錯誤訊息

            ```txt
            0/3 nodes are available: 3 node(s) didn't find available persistent volumes to bind.
            ```

        - 錯誤截圖

            ![7pvfail](https://user-images.githubusercontent.com/3851540/63025858-a9a8b800-bedc-11e9-9fbc-ea1ec8459428.png)

## 使用 KubeDB 安裝 Redis Cluster

1. 建立 `demo` kubernetes namespace

    > 可忽略，但後續動作 yaml 中的 namespace 記得要改

    ```bash
    kubectl create ns demo
    ```

2. 建立 `redis-cluster.yaml`

    ```yaml
    apiVersion: kubedb.com/v1alpha1
    kind: Redis
    metadata:
      name: redis-cluster
      namespace: demo
    spec:
      version: 5.0.3-v1
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

3. 使用 `redis-cluster.yaml` 建立 Redis

    ```bash
    kubedb create -f redis-cluster.yaml
    ```

4. 確認成功安裝

    - 成功建立 三 個 StatefulSet 且各有 二 個 pod 運行中

        ```bash
        kubedb describe rd -n demo redis-cluster
        ```

        ![8statefulsetok](https://user-images.githubusercontent.com/3851540/63025859-a9a8b800-bedc-11e9-8da8-4cce376fca18.png)

    - 連線至 redis 中確認 cluster 狀態

        ```bash
        kubectl -n demo exec -it redis-cluster-shard0-0 redis-cli cluster nodes
        ```

        ![9clusternodes](https://user-images.githubusercontent.com/3851540/63025861-a9a8b800-bedc-11e9-969e-861d15405ca3.png)

## 移除 KubeDB

1. 下載 sh 並執行清除所有 object

    ```bash
    curl -fsSL https://raw.githubusercontent.com/kubedb/cli/0.12.0/hack/deploy/kubedb.sh | bash -s -- --uninstall --namespace=kube-system --purge
    ```

2. 移除 KubeDB operator chart

    ```bash
     helm del kubedb-operator --purge --no-hooks
    ```

3. 移除 KubeDB catalog

    ```bash
    helm del kubedb-catalog --purge --no-hooks
    ```

4. 移除沒用到的 PVC

    ```bash
    kubectl get pvc -n demo --no-headers -o custom-columns=":metadata.name" | xargs -n 1 kubectl -n demo delete pvc
    ```

## 心得

本來就沒有 Kubernetes 的 production 相關經驗，一下跳到 Helm 不打緊，還用上了 Operator，還好同事很強大，讓我越級打怪還沒遇到解決不了的問題  讚啦 ~~~ 

回到這次透過 KubeDB 安裝 Redis Cluster 上，一開始依網站上的 Helm 安裝步驟操作總是遇到問題，同事建議改用網站上的 custom install script 停用 webhook 才過了第一關，直到後來在紀錄安裝步驟時才發現 Helm 其實也有提供停用 webhook 的參數設定，只是網站上沒提到，參數設定說明是在 GitHub 上翻到的，類似的情況還發生在一些 yaml 設定，官網上的說明文件有時比較新、比較正確，不過偶爾會反過來，GitHub 比較正確，就這樣兩邊參照、比對、嘗試可是吃盡了苦頭呀

除此之外不知道是我的基礎知識不足還是官網的文件東漏西漏，我個人是踩了不少雷才試出個正確的流程，而同事使用 Minikube 到是沒遇到什麼問題 XD，我人品真這麼差嗎 ~~~

## 參考資訊

1. [Installation Guide](https://kubedb.com/docs/0.12.0/setup/install/)
2. [Kubernetes (5) Local Persistent Volumes – A Step-by-Step Tutorial](https://vocon-it.com/2018/12/20/kubernetes-local-persistent-volumes/#Step_1_Create_StorageClass_with_WaitForFirstConsumer_Binding_Mode)
3. [KubeDB - Redis Cluster](https://kubedb.com/docs/0.12.0/guides/redis/clustering/redis-cluster/)
4. [kubedb/installer](https://github.com/kubedb/installer/tree/0.12.0/chart/kubedb)
5. [Uninstall KubeDB](https://kubedb.com/docs/0.12.0/setup/uninstall/)
