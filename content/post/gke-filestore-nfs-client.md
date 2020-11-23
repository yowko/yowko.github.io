---
title: "使用 gcloud shell 建立 GKE 並使用 Filestore 做為 NFS-Client"
date: 2020-11-23T21:30:00+08:00
lastmod: 2020-11-23T21:30:31+08:00
draft: false
tags: ["GCP","Kubernetes"]
slug: "gke-filestore-nfs-client"
---

## 使用 gcloud shell 建立 GKE 並使用 Filestore 做為 NFS-Client

許多目前用到的 service 都需要使用 storageclass 來動態建立 volume，雖然使用預設的 `standard` 可以順利讓 service 啟動，但為了保存相關資料還是需要使用外部 disk，今天就紀錄一下將 Filestore mount 給 GKE 做 nfs

## 基本環境說明

1. macOS Catalina 10.15.7
2. gcloud SDK 319.0.0

## 設定方式

1. 啟用 file Google APIs

    > 未啟用無法使用 `gcloud` 建立 filestore

    ```bash
    gcloud services enable file.googleapis.com
    ```

    ![1notenablefileapi](https://user-images.githubusercontent.com/3851540/99950083-f739f300-2db6-11eb-8611-75b32ed269dc.png)

2. 安裝 `beta` gcloud components

    > 未安裝 `beta` gcloud components 無法建立 filestore
    ![3betacomponent](https://user-images.githubusercontent.com/3851540/99951382-015cf100-2db9-11eb-922d-5cc44db64936.png)

    ```bash
    gcloud components install beta
    ```

3. 建立 Filestore

    > {SIZE} 至少要 1024GB(1TB)
    ![21024gb](https://user-images.githubusercontent.com/3851540/99950087-facd7a00-2db6-11eb-991e-083471275a96.png)

    - 語法

        ```bash
        gcloud beta filestore instances create {FILESTORE NAME} \
            --project={PROJECT NAME} \
            --zone={ZONE ID} \
            --tier=STANDARD \
            --file-share=name="volumes",capacity={SIZE} \
            --network=name="default"
        ```

    - 範例

        ```bash
        gcloud beta filestore instances create testnfs \
        --project=testproject \
        --zone=asia-east2-a \
        --tier=STANDARD \
        --file-share=name="volumes",capacity=1TB \
        --network=name="default"
        ```

4. 取得 Filestore ip

    > 先將 ip 紀錄下來，後面會用到

    - 語法

        ```bash
        echo $(gcloud beta filestore instances describe {FILESTORE NAME} \
        --project={PROJECT NAME} \
        --zone={ZONE ID} \
        --format="value(networks.ipAddresses[0])")
        ```

    - 範例

        ```bash
        echo $(gcloud beta filestore instances describe testnfs \
        --project=testproject \
        --zone=asia-east2-a \
        --format="value(networks.ipAddresses[0])")
        ```

5. 建立 GKE instance 並設定權限 (已有 GKE instance 可略過)

    - 5-1. 建立 GKE instance

        - 語法

            ```bash
            gcloud container clusters create {gke name}
            ```

        - 範例

            ```bash
            gcloud container clusters create testk8s
            ```

    - 5.2. 取得 gke 憑證

        - 語法

            ```bash
            gcloud container clusters get-credentials {gke name}
            ```

        - 範例

            ```bash
            gcloud container clusters get-credentials testk8s
            ```

    - 5.3. 設定 cluster-admin 權限

        ```bash
        ACCOUNT=$(gcloud config get-value core/account)
        kubectl create clusterrolebinding core-cluster-admin-binding \
            --user ${ACCOUNT} \
            --clusterrole cluster-admin
        ```

6. 安裝 Helm

    ```bash
    brew install helm
    ```

7. 安裝 nfs-client

    > 將前面紀錄下來的 ip 填入

    - 語法

        ```bash
        helm install  nfs-cp --set nfs.server={FILESTORE IP} --set nfs.path=/volumes stable/nfs-client-provisioner
        ```

    - 範例

        ```bash
        helm install  nfs-cp --set nfs.server=10.94.204.42 --set nfs.path=/volumes stable/nfs-client-provisioner
        ```

8. 移除預設 `standard` 為 default 並將 nfs-client 設為 default storageclass

    ```bash
     kubectl patch storageclass standard -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}' && kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

- 指定 nfs-client 為 storageclass 的 helm 安裝語法

    ```bash
    helm repo add hashicorp https://helm.releases.hashicorp.com && helm install consul hashicorp/consul --set global.name=consul --set StorageClass=nfs-client
    ```

## 心得

絕大部操作都與 [Dynamically provision GKE storage from Filestore using the NFS-Client provisioner](https://cloud.google.com/community/tutorials/gke-filestore-dynamic-provisioning) 相同，其中只有下列幾點不同

1. helm 的安裝： mac 上就直接使用 brew 安裝 3.X 版本 (文件則是使用壓縮檔安裝 2.11 版)
2. rbac 設定：helm 3 之後 tiller 已不再需要，故未設定 tiller 用的 service account
3. helm 語法調整：helm 3 語法與 helm 2 略有不同 (helm 3 移除 `--name` flag)

## 參考資訊

1. [Dynamically provision GKE storage from Filestore using the NFS-Client provisioner](https://cloud.google.com/community/tutorials/gke-filestore-dynamic-provisioning)
2. [Installing Helm](https://helm.sh/docs/intro/install/)
3. [hashicorp/consul-helm](https://github.com/hashicorp/consul-helm)
4. [Change the default StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)
