---
title: "透過 Kubespray 在 Kubernetes 上安裝 Helm"
date: 2019-07-14T21:30:00+08:00
lastmod: 2019-07-14T21:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "kubespray-helm"
---

## 透過 Kubespray 在 Kubernetes 上安裝 Helm

經過一段時間的發展 Kubernetes 目前是 container 調度工具中最受重視的，不過 Kubernetes 只是用來管理 container 的調度，該如何決定調度的計劃以及內容就成了新課題，而 Helm 就是其中的解決方式之一

Helm 是 Kubernetes 中用來管理 Chart (package) 的工具，其中 Chart 則是用來描述 Kubernetes package 的 yml 檔案，Relase 則是透過 Chart 執行的部署

之前筆記 [透過 Kubespray 來架設 Kubernetes](https://blog.yowko.com/kubespray-kubernetes) 紀錄到該如何使用 Kubespray 來架設 Kubernetes，今天就來紀錄一下如何使用 Kubespray 安裝 Helm 吧

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
6. Kubernetes cluster 內容請參考之前筆記 [透過 Kubespray 來架設 Kubernetes](https://blog.yowko.com/kubespray-kubernetes)

## 安裝 Helm

Kubespray 在相關套件：dashboard、helm、cni 的支援度很好，很多功能都可以透過 yml 的修改搭配簡易的指令就可以完成

1. 調整 `inventory/{name}/group_vars/k8s-cluster/addons.yml`

    > 延續之前內容，使用 `inventory/k8s/group_vars/k8s-cluster/addons.yml` 為例

    ```yml
    helm_enabled: true
    ```

2. 執行部署

    ```bash
    ansible-playbook -i inventory/k8s/inventory.ini cluster.yml -b -v -k
    ```

## 心得

目前執行 helm 的部署還是透過建立 kubernetes claster 的腳本，其實就是再一次執行架設 kubernetes 的流程，我暫時還沒找到方法可以讓整個執行流程加快，但不用自行手動安裝 helm 還是挺不賴的，yml 調一調，執行一句指令就可以輕鬆搞定

不過整個安裝流程跟版本以及相關設定都被包在 Kubespray 的腳本中，如果想要做些細微調整，還是得要好好花些時理解跟上手 Ansible 才行

## 參考資訊

1. [[Kubernetes] Package Manager - Helm 簡介](https://godleon.github.io/blog/Kubernetes/k8s-Helm-Introduction/)
2. [透過 Kubespray 來架設 Kubernetes](https://blog.yowko.com/kubespray-kubernetes)
