---
title: "在 Kubernetes 上安裝 cert-manager"
date: 2021-02-08T20:30:00+08:00
lastmod: 2021-02-08T20:30:31+08:00
draft: false
tags: ["Kubernetes","Helm"]
slug: "cert-manager"
---

## 在 Kubernetes 上安裝 cert-manager

之前 gRPC 都是透過 insecure 的設定來避免 gRPC 需要 SSL 的問題，雖然使用上並沒有遇到什麼狀況，但還是覺得不太正統，另外程式碼也多了一些設定，所以一直想嘗試讓內部 gRPC 也使用 SSL，要使用 SSL 首先就是需要一張憑證，今天就紀錄一下如何安裝 Kubernetes 上的憑證管理工具 ： `cert-manager`

之前曾經考慮過將 Kubernetes 既有的憑證 mount 給 application 做使用，但後來因為不知道 Kubernetes 既有憑證的更新機制，擔心既有憑證用一用，Kubernetes 更新憑證 application 未能即時同步更新造成異常，所以最後決定另外自簽憑證

cert-manager 是基於 Kubernetes 的 open source 憑證管理工具，可以協助從包含 `Let's Encrypt`, `HashiCorp Vault`, `Venafi`,與`自簽憑證`等多個不同來源來發行憑證

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0(51484)

    - docker engine 20.10.2
    - Kubernetes v1.19.3

3. Helm chart

    - cert-manager v1.1.0

## 使用 Helm 安裝

> 有其他安裝方式，有興趣的可以參考 [Welcome to cert-manager/Installation/Kubernetes](https://cert-manager.io/docs/installation/kubernetes/#installing-with-regular-manifests)

1. 建立 namespace

    ```bash
    kubectl create namespace cert-manager
    ```

2. 加入 Helm repository

    ```bash
    helm repo add jetstack https://charts.jetstack.io && helm repo update
    ```

3. 安裝 cert-manager

    ```bash
    helm install \
    cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v1.1.0 \
    --set installCRDs=true
    ```

4. 驗證安裝

    ```bash
    kubectl get pods --namespace cert-manager
    ```

    ![1crdverify](https://user-images.githubusercontent.com/3851540/107198823-8f4b8e00-6a30-11eb-8ca7-a59c842749f4.png)

## 心得

cert-manager 的安裝方式，嚴格說來有三種

1. 使用 yaml 安裝 cert-manager
2. 使用 yaml 安裝 CRS 搭配 Helm 安裝 cert-manager
3. 使用 Helm 安裝 CRD 與 cert-manager

我沒有刻意去比較用法差異，我懶所以直接選用 `3. 使用 Helm 安裝 CRD 與 cert-manager` 一句 script 可以裝到好，雖然 `使用 yaml 安裝 cert-manager` 也是一句，但我自己執行時是卡住滿久的，最後等不下去就 cancel 了，大家挑自己順眼的就好

## 參考資訊

1. [Welcome to cert-manager](https://cert-manager.io/docs/)
2. [使用 cert-manager 管理 K8S TLS 憑證](https://medium.com/starbugs/%E4%BD%BF%E7%94%A8-cert-manager-%E7%AE%A1%E7%90%86-k8s-tls-%E6%86%91%E8%AD%89-ab6258af9195)
3. [Welcome to cert-manager/Installation/Kubernetes](https://cert-manager.io/docs/installation/kubernetes/#installing-with-regular-manifests)
