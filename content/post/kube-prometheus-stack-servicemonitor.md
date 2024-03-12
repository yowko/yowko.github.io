---
title: "使用 Helm Chart kube-prometheus-stack 透過 ServiceMonitor 自動蒐集 metrics"
date: 2024-04-10T00:30:00+08:00
lastmod: 2024-04-10T00:30:31+08:00
draft: false
tags: ["helm","kubernetes","prometheus","grafana","mysql","metrics","monitoring"]
slug: "kube-prometheus-stack-servicemonitor"
---

## 使用 Helm Chart kube-prometheus-stack 透過 ServiceMonitor 自動蒐集 metrics

過去團隊會在建立 storage service 連帶建立相對應的 exporter，以 MySql 為例，ansible 的腳本已經包含建立 mysql service 與 mysql exporter service，並在 mysql 與 mysql exporter 啟動後再註冊相關 prometheus 的 scrape config 中加入相對應的 job，這樣的方式在服務數量增加時，會有一定的維護成本，且不易管理。加上團隊的 prometheus 本來就在 kubernetes 上，本來就可以將 exporter 安裝進 kubernetes，所以就來試試看使用 Helm Chart kube-prometheus-stack 透過 ServiceMonitor 自動蒐集 metrics。

今天會以 mysql-exporter 做為範例，來說明如何設定 kube-prometheus-stack 與 prometheus-mysql-exporter 之間的關聯

## 基本環境說明

1. macOS Sonoma 14.3.1 (Apple M2 Pro)
2. OrbStack Version 1.4.3 (16673)
3. Helm version.BuildInfo{Version:"v3.14.2", GitCommit:"c309b6f0ff63856811846ce18f3bdc93d2b4d54b", GitTreeState:"clean", GoVersion:"go1.22.0"}
4. Helm Charts

    - prometheus-community/kube-prometheus-stack-57.0.1
    - prometheus-community/prometheus-mysql-exporter-2.5.0

5. Docker images

    - mysql:8.3.0

6. 使用 docker 建立測試用 MySQL

    ```bash
    docker run --rm --name mysql -p 3306:3306 -e MYSQL_USER=monitoring -e MYSQL_PASSWORD=pass.123 -e MYSQL_ROOT_PASSWORD=pass.123 -d mysql:8.3.0
    ```

## 設定方式

1. 新增 Helm repository

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    ```

2. 使用 Helm 安裝 kube-prometheus-stack

    > 這是一般步驟，需要留意的是 release name (以下的例子是`kube-prometheus-stack`) 後面會用到

    ```bash
    helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack
    ```

3. 使用 Helm 安裝 prometheus-mysql-exporter

    ```bash
    helm install --set serviceMonitor.enabled=true --set serviceMonitor.additionalLabels.release=kube-prometheus-stack --set mysql.host=host.docker.internal --set mysql.user=monitoring --set mysql.pass=pass.123 mysql-exporter prometheus-community/prometheus-mysql-exporter
    ```

    - `serviceMonitor.enabled=true` 代表啟用 ServiceMonitor
    - `serviceMonitor.additionalLabels.release=kube-prometheus-stack` 代表將 ServiceMonitor 會增加 kube-prometheus-stack release name 的 label 讓 prometheus operator 可以自動探查
    - `mysql.host=host.docker.internal` 代表 mysql 服務的位置為 `host.docker.internal`，這是 docker for mac 的特殊設定
    - `mysql.user=monitoring` 代表 mysql 服務的使用者為 `monitoring` (這是基本環境說明中 mysql 啟動時設定加入的 user)
    - `mysql.pass=pass.123` 代表 mysql 服務的密碼 (這是基本環境說明中 mysql 啟動時設定的 password)

## 心得

- 安裝 mysql exporter 前

    ![1before](https://github.com/yowko/picsbed/assets/3851540/4215e26d-16f5-4155-8f1b-fd98dd827959)

- 安裝 mysql exporter 後

    ![2after](https://github.com/yowko/picsbed/assets/3851540/97028251-733b-423f-ae36-d8eef5fc49ec)

我查了一些資料，發現大多數還是都使用 prometheus 的 scrape config 來設定，但這樣的幾個缺點：

1. 相依性問題

    > prometheus 與 exporter 安裝順序會互卡：prometheus 安裝時，mysql exporter 尚未安裝，所以 scrape config 還不能設定，變成 mysql exporter 安裝後再額外調整 scrape config，流程變得複雜

2. 維護成本高

    > 因為相依問題，所以設定時為了確保正確性就會多了不少步驟或是流程，無形中會增加維護成本

不知道是我對 ServiceMonitor 認識有限的關係，還是我漏看了文件，我花了一整天查為什麼 prometheus 沒有自動將 mysql exporter 的資料收進 prometheus 的狀況，最後才發現 `serviceMonitor.additionalLabels.release` 需要設定成 kube-prometheus-stack 的 release name

## 參考資訊

1. [GitHub - charts/kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
2. [GitHub - charts/prometheus-mysql-exporter](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-mysql-exporter)
