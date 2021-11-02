---
title: "使用 Kubespray 開啟 Kubernetes Dashboard"
date: 2019-08-10T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "kubespray-kubernetes-dashboard"
---

## 使用 Kubespray 開啟 Kubernetes Dashboard

前同事問到 Kubernetes Dashboard 安裝的問題，這才發現我只用過別人裝好的，自己沒有真的裝過 XD，趁著假日時間紀錄一下囉

Kubernetes Dashboard 是 Kubernetes 官方製作用來管理 Kubernetes clusters 的 web-base UI 工具

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

3. Kubespray 2.10
4. Kubernetes 1.14.3

## 安裝 Kubernetes Dashboard

透過 kubespray 安裝 Kubernetes Dashboard 非常簡單，而且預設就會執行安裝

1. 確認是否啟用安裝

    > 檢查 `kubespray/inventory/k8s/group_vars/k8s-cluster/addons.yml` 中的 `dashboard_enabled` 設定是否為 `true`

    ![1enable](https://user-images.githubusercontent.com/3851540/62828782-eb0d3f00-bc20-11e9-8d2b-236b525db478.png)

2. 檢查是否已成功安裝

    ```bash
    kubectl -n kube-system get service kubernetes-dashboard
    ```

3. 修改 Kubernetes Dashboard service 設定

    ```bash
    kubectl -n kube-system edit service kubernetes-dashboard
    ```

    - 指定 nodePort
    - 將 type 由 ClusterIP 改為 NodePort

    > 修改前

    ![2before](https://user-images.githubusercontent.com/3851540/62828783-eba5d580-bc20-11e9-9632-1a7b684a5563.png)

    > 修改後

    ![3after](https://user-images.githubusercontent.com/3851540/62828784-eba5d580-bc20-11e9-870a-dd128b76f467.png)

4. 開啟 `https://node1:30710` 即可出現登入畫面

    ![4login](https://user-images.githubusercontent.com/3851540/62828785-eba5d580-bc20-11e9-8de4-1d6b764e0cf0.png)

## 設定 Kubernetes Dashboard 登入資訊

> 畫面上可以使用 `kubeconfig` 或是 `token` 登入，以下使用 `token` 示範囉 (我承認 `kubeconfig` 太複雜，我搞不定XD)

1. 建立 admin-role.yaml

    > 建立 `admin` 為管理者

    ```yaml
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
    name: admin
    annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
    roleRef:
    kind: ClusterRole
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
    subjects:
    - kind: ServiceAccount
    name: admin
    namespace: kube-system
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: admin
    namespace: kube-system
    labels:
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    ```

2. 建立 Service Account 與角色綁定

    ```bash
    kubectl create -f admin-role.yaml
    ```

3. 取得 admin-token 的 secret 名字

    ```bash
    kubectl -n kube-system get secret|grep admin-token
    ```

    ![5secretname](https://user-images.githubusercontent.com/3851540/62828786-ec3e6c00-bc20-11e9-9462-ef2afad057d8.png)

4. 取得 token

    > `admin-token-h2dxb` 是上一個步驟的結果，每次建立都會不同，記得替換

    ```bash
    kubectl -n kube-system get secret admin-token-h2dxb -o jsonpath={.data.token}|base64 -d
    ```

    > 以下語法可以看到 secret 較多資訊

    ```bash
    kubectl -n kube-system describe secret admin-token-h2dxb
    ```

5. 成功登入

    ![5afterlogin](https://user-images.githubusercontent.com/3851540/62828787-ec3e6c00-bc20-11e9-858e-3f2ea98d48f3.png)

## 心得

透過 Kubespray 原則上沒有安裝的問題，預設就啟用了 Kubernetes Dashboard 的安裝，只要 Kubernetes 安裝成功，Kubernetes Dashboard 就會部署成功，我個人覺得登入的相關設定反而還比較困難些 XD

## 參考資訊

1. [kubernetes/dashboard](https://github.com/kubernetes/dashboard)
2. [使用 kubeconfig 或 token 进行用户身份认证](https://rootsongjc.gitbooks.io/kubernetes-handbook/guide/auth-with-kubeconfig-or-token.html)
