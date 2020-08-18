---
title: "kubectl patch 錯誤"
date: 2020-08-18T21:30:00+08:00
lastmod: 2020-08-18T21:30:31+08:00
draft: false
tags: ["kubernetes"]
slug: "kubectl-patch-fail"
---

## kubectl patch 錯誤

這件事實在太瞎了，不得不紀錄一下

這幾天正在進行團隊各環境的 service 升級，其中的重點就是 kubernetes 版本，重新 refactor 所有 ansible scripts 後，終於沒有出錯，但在部署時都一直出現某些 service 起不來，細查後發現 pvc 在 bind pv 時沒有成功，再進一步追查發現是沒有預設的 storageclass 造成的 (也就是在將 storageclass 設定成 default 的出現異常)，就來看看到底發生什麼事了

## 基本環境說明

1. CentOS Linux 7
2. Kubernetes 1.17.8
3. Kubectl

    - Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.8", GitCommit:"35dc4cdc26cfcb6614059c4c6e836e5f0dc61dee", GitTreeState:"clean", BuildDate:"2020-06-26T03:43:27Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
    - Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.8", GitCommit:"35dc4cdc26cfcb6614059c4c6e836e5f0dc61dee", GitTreeState:"clean", BuildDate:"2020-06-26T03:36:03Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}

## 錯誤訊息一

1. 執行指令

    ```bash
    kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

2. 錯誤訊息

    ```txt
    error: there is no need to specify a resource type as a separate argument when passing arguments in resource/name form (e.g. 'kubectl get resource/<resource_name>' instead of 'kubectl get resource resource/<resource_name>'
    ```

3. 錯誤截圖

    ![1error1](https://user-images.githubusercontent.com/3851540/90493193-a2ae2880-e174-11ea-8848-6fc3cde5f2b1.png)

## 錯誤訊息二

> 根據上述的錯誤訊息將 `storageclass nfs-client` 改為 `storageclass/nfs-client`

1. 執行指令

    ```bash
    kubectl patch storageclass/nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

2. 錯誤訊息

    ```txt
    error: the server doesn't have a resource type "{\"annotations\":{\"storageclass"
    ```

3. 錯誤截圖

    ![2error2](https://user-images.githubusercontent.com/3851540/90493201-a5108280-e174-11ea-83d8-10703bf242a2.png)

## 解決方式

- 可用指令

    ```bash
    kubectl patch storageclass/nfs-client -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

- 異常指令

    ```bash
    kubectl patch storageclass/nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

看不出差別？  讓我將兩個指令放在一起看，就是差一個空白空元

![3compare](https://user-images.githubusercontent.com/3851540/90493203-a5a91900-e174-11ea-8e7e-0b7d703fe41c.png)

## 心得

三更半夜卡在這，心裡的憤怒值倍增呀，加上這句 script 是從官網上 copy 來的，讓我不得不筆記一篇發洩一下

官網有錯誤指令並不少見，錯誤訊息看不出問題也不算特別異常，一個空白字元造成問題也還能接受，但三個情境撞在一起  實在是難搞呀

我另外試了不同版本的 kuectl 就沒遇到這個問題

- Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.5", GitCommit:"e6503f8d8f769ace2f338794c914a96fc335df0f", GitTreeState:"clean", BuildDate:"2020-06-26T03:47:41Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"darwin/amd64"}
- Server Version: version.Info{Major:"1", Minor:"16+", GitVersion:"v1.16.6-beta.0", GitCommit:"e7f962ba86f4ce7033828210ca3556393c377bcc", GitTreeState:"clean", BuildDate:"2020-01-15T08:18:29Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"})

## 參考資訊

1. [Change the default StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)
