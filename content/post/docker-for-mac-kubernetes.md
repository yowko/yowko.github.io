---
title: "在 Docker for Mac 上啟用 Kubernetes"
date: 2019-07-21T21:30:00+08:00
lastmod: 2019-07-21T21:30:31+08:00
draft: false
tags: ["Mac","Docker","Kubernetes"]
slug: "docker-for-mac-kubernetes"
---

## 在 Docker for Mac 上啟用 Kubernetes

現在工作的部暑都是透過 Kubernetes 來執行，不過我還是常常會卡東卡西，又不能老是使用團隊環境來練習，加上工作內部的資料也無法部署至自己建的雲端 Kubernetes cluster 中，所以正在嘗試可以在 local 建立 Kubernetes cluster 的方式

今天就先簡單筆記一下如何使用 Docker for Mac 內建的 Kubernetes 功能

## 基本環境說明

1. macOS Mojave 10.14.5
2. Docker Desktop 2.0.0.3 (31259)

    - Docker Engine 18.09.2
    - Kubernetes v1.10.11

## 關於 Swarm 與 Kubernetes

- Swarm 是 Docker 公司自行開發的 container 調度工具，也因此很早期就整合進 docker engine 之中，不需要額外安裝套件即可使用
- Kubernetes 為 Google open source 的 container 調度工具，架構較為彈性而較受到市場歡迎，也因此讓 Docker 公司將其整合進 Docker 中

目前只有 `Docker Desktop for Mac 18.06.0-ce-mac70 CE` 之後版本可以支援 Kubernetes

## 啟用 Kubernetes

1. 點 `Docker Desktop` 圖示 --> `Preferences`

    ![1preferemces](https://user-images.githubusercontent.com/3851540/61593260-dafcd380-ac0f-11e9-9648-1050b2bbafcd.png)

2. Kubernetes --> Enable Kubernetes --> Apply

    ![2enablek8s](https://user-images.githubusercontent.com/3851540/61593261-dafcd380-ac0f-11e9-92db-2ea958be1568.png)

3. Install Now

    ![3install](https://user-images.githubusercontent.com/3851540/61593262-dafcd380-ac0f-11e9-8e60-02668a96f0be.png)

4. Kubernetes is Starting

    ![4installing](https://user-images.githubusercontent.com/3851540/61593263-db956a00-ac0f-11e9-9080-58ae14163cfe.png)

5. Kubernetes is running

    ![5running](https://user-images.githubusercontent.com/3851540/61593264-db956a00-ac0f-11e9-9e29-411f4721df5d.png)

6. 確認執行中

    ![6running](https://user-images.githubusercontent.com/3851540/61593265-db956a00-ac0f-11e9-8819-073b14dd7ce4.png)

    ![7running](https://user-images.githubusercontent.com/3851540/61593266-db956a00-ac0f-11e9-9539-2abc738f0b61.png)

## 心得

紀錄完才發現原來在 Docker for Mc 上啟用 Kubernetes 如此簡單，不得不說整合得相當好呀XD

本來在考慮這麼虛的筆記內容要不要上傳，不過既然都截圖了，還是放一下備忘囉

## 參考資訊

1. [Deploy on Kubernetes](https://docs.docker.com/docker-for-mac/kubernetes)
2. [Tutorial : Getting Started with Kubernetes with Docker on Mac](https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd)
