---
title: "在 Kubernetes 上使用 Helm 安裝 Istio"
date: 2019-07-14T22:30:00+08:00
lastmod: 2019-07-14T22:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "Kubernetes-helm-istio"
---

## 在 Kubernetes 上使用 Helm 安裝 Istio

其實對於 Istio 到底解決了什麼問題，我沒有把握可以講得很清楚，就我粗淺地理解是為了更有效地簡化 Pod 間的溝通與管理，但好壞都要等實際體驗過才說得準，所以先來紀錄一下如何在 Kubernetes 透過 Helm 來安裝 Istio，待我實際使用過後再來分享優缺點

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
4. Kubernetes 2.14.3
5. Helm v2.13.1
6. Istio 1.2.2
7. Kubernetes cluster 內容請參考之前筆記 [透過 Kubespray 來架設 Kubernetes](https://blog.yowko.com/kubespray-kubernetes) 與 [透過 Kubespray 在 Kubernetes 上安裝 Helm](https://blog.yowko.com/kubespray-helm)

## 安裝 Istio

1. 下載並解壓 Istio

    ```bash
    curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.2 sh -
    ```

2. 建立 service account for Tiller

    ```bash
    cd istio-1.2.2

    kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
    ```

3. 安裝 Tiller

    ```bash
    helm init --service-account tiller
    ```

4. 安裝 `istio-init` 來啟動 Istio 的所有 CRD

    ```bash
    helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
    ```

5. 驗證是否有 `23` CRD 被加至 Kubernetes 的 api-server 中

    ```bash
    kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l
    ```

    > 執行結果若為 `23` 就是正確

6. 安裝 istio

    ```bash
    helm install install/kubernetes/helm/istio --name istio --namespace istio-system
    ```

## 心得

安裝上難度不高，但在確認正確性卻不如想像中容易

1. 確認 service 是否完整部署

    ```bash
    kubectl get svc -n istio-system
    ```

    > 可以跟 [Installation Configuration Profiles](https://istio.io/docs/setup/kubernetes/additional-setup/config-profiles/) 比對一下，是否對應的 service 都有被部署，以    `default` 為例總共需要 `8` 個 service

2. 確保 Kubenetes 的 Pod 都是 `Running`

    ```bash
    kubectl get pods -n istio-system
    ```

    > 經過數次安裝，我一直無法得到所有 Pod 都是 Running：固定三個 `istio-init-crd-*` 的狀態都是 `Completed`，同事解釋那是啟動用的，執行階段不影響，幸虧有同事指導，不然我不知道還要在這上面卡多久，嘗試各種安裝方式都是相同結果呀XD

對於 Istio 目前我還在學習安裝階段，不到實際使用，待日後有真正實際經驗再來紀錄相關內容

## 參考資訊

1. [為什麼我們需要Istio？](https://jimmysong.io/posts/why-do-we-need-istio/)
2. [Service Mesh與Istio佈署簡介(上)](https://www.inwinstack.com/2018/01/12/service-mesh-istio/)
3. [Istio](https://istio.io/)
4. [Customizable Install with Helm](https://istio.io/docs/setup/kubernetes/install/helm/)
