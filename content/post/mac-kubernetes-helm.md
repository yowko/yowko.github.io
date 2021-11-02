---
title: "在 Docker for Mac 啟用 Kubernetes 後安裝 Helm"
date: 2019-12-22T10:30:00+08:00
lastmod: 2021-11-02T10:30:31+08:00
draft: false
tags: ["macOS","Tools","Helm","Kubernetes","Docker"]
slug: "docker-mac-kubernetes-helm"
---

## 在 Docker for Mac 啟用 Kubernetes 後安裝 Helm

在之前筆記 [在Docker for Mac 上啟用Kubernetes](/docker-for-mac-kubernetes/) 中曾經紀錄到如果想在 mac 中嘗試 Kubernetes 可以透過 Docker Desktop for Mac 的內建功能來啟用 Kubernetes

過去測試 Helm 相關功能都會透過在雲端自建 Kubernettes instance 來進行，雖然已經非常便利，但不免還是需要在不同 server 間做轉換，於是興起了在 Docker for Mac 的 Kubernetes instance 中執行 Helm 的念頭，雖然這個念頭不是第一次有，但之前因為安裝出現錯誤加上急著要解決專案問題，沒有花時間在排除安裝錯誤的問題上，這次又遇到相同狀況，筆記一下

## 基本環境說明

1. macOS Catalina 10.15.1
2. Docker Desktop for Mac community 2.1.0.5(40692)
3. Kubernetes v1.14.8

## 安裝方式

> 請依需要挑選要安裝的版本

- 安裝目前最新版本 `3.0.2`

    ```bash
    brew install helm
    ```

- 安裝特定版本

    > 詳細內容可以參考之前筆記 [使用 Homebrew 安裝舊版 Helm](/brew-install-specific-version-helm)

  - 取得版本列表

    ```bash
    brew search helm@
    ```

  - 安裝指定版本 `helm@2`

    ```bash
    brew install helm@2
    ```

## 錯誤排除

1. 以下狀況請重設 Kubernetes

    - 錯誤訊息

        - 執行 `helm version` 出現以下錯誤訊息

            ```txt
            Client: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
            Error: Get https://localhost:6443/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller: dial tcp [::1]:6443: connect: connection refused
            ```

            ![1helmversionerror](https://user-images.githubusercontent.com/3851540/71316548-efd5df80-24ab-11ea-9d09-565fdbda1302.png)

        - 執行 `kubectl get nodes` 出現以下錯誤訊息

            ```txt
            Error: Get https://localhost:6443/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller: dial tcp [::1]:6443: connect: connection refused
            ```

            ![2kubectlgeterror](https://user-images.githubusercontent.com/3851540/71316549-efd5df80-24ab-11ea-93cb-af9c4c6f4830.png)

    - 重設步驟

        1. 刪除 Kubernetes 所有設定檔

            ```bash
            rm -rf ~/.kube
            ```

        2. 重啟 docker 和 Kubernetes

2. 以下狀況請重新啟用 Kubernetes

    - 執行 `kubectl get nodes` 出現以下錯誤訊息

        ```txt
        The connection to the server localhost:8080 was refused - did you specify the right host or port?
        ```

    - 啟用 Kubernetes 方式

        ![dockerformac](https://user-images.githubusercontent.com/3851540/61593260-dafcd380-ac0f-11e9-9648-1050b2bbafcd.png)

        ![enablekubenetes](https://user-images.githubusercontent.com/3851540/61593261-dafcd380-ac0f-11e9-92db-2ea958be1568.png)

## 心得

不知道問題發原因為何，但兩次嘗試在 Docker for Mac 上安裝 Helm 都遇到相同問題，而這也讓我相當擔心穩定性：會不會花了許多時間建置環境，測試到一半就毀損需要重來，不過空擔心也沒用，試試看最清楚了

## 參考資訊

1. [在Docker for Mac 上啟用Kubernetes](/docker-for-mac-kubernetes/)
2. [使用 Homebrew 安裝舊版 Helm](/brew-install-specific-version-helm)
3. [Mac Kubernetes 报 The connection to the server localhost:6443 was refused](https://blog.csdn.net/iaiot/article/details/85218642)
