---
title: "使用 dnf 透過 module 安裝指定版本套件"
date: 2020-10-23T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["Linux"]
slug: "dnf-module-install-specific-artifact"
---

## 使用 dnf 透過 module 安裝指定版本套件

這是從過去使用 `yum` 時發現的問題：安裝套件時多是執行 `yum install -y {套件名稱}`，如此一來會直接安裝該套件的最新版本，對於日常使用或是全新開發上是正確的，但這個動作對於正在運作的系統，這個直接動作會造成原本的套件升級至最新版，可能出現不可預期的問題，所以之前筆記 [yum 安裝指定版本套件](/yum-specific-version/) 就紀錄了該如何使用 yum 來安裝指定版本套件的做法 (指定版本並不一定是舊版，有可能指定的版本正好是最新版，只是確保版本是固定的)，最近正在將 yum 改為 dnf，在 dnf module 安裝時沒辦法直接套用相同技巧，立馬簡單紀錄一下

## 基本環境說明

1. Azure VM Standard_B2s (CentOS-based 8.2 - Gen1)
2. Linux (centos 8.2.2004)
3. Linux Kernel 4.18.0-193.19.1.el8_2.x86_64
4. Redis 6.0.7

## 操作步驟

> 這邊以 redis 為例，想多了解如何使用 dnf 安裝 redis 可以參考之前筆記 [使用 dnf 安裝 Redis 6](/dnf-install-redis6/)

1. 確保目標 package 的 repo rpm 已安裝

    ```bash
    dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
    ```

2. 依 module 列出可用的 package 與 stream

    ```bash
    dnf module list redis
    ```

    ![1dnflist](https://user-images.githubusercontent.com/3851540/96956251-395ae500-152a-11eb-84be-e096dd132319.png)

3. 檢視目標 package 的詳細資訊

    > 確認是否有無不同版本的 artifacts

    - 語法

        > 如果只有一個 `profile` 可以忽略不寫

        ```bash
        dnf module info {package Name}:{Stream}/{profile}
        ```

        ![2dnfinfo](https://user-images.githubusercontent.com/3851540/96956255-3bbd3f00-152a-11eb-8d52-6d6a2c01e3f9.png)

    - 範例

        ```bash
        dnf module info redis:remi-6.0
        ```

        ![3noprofile](https://user-images.githubusercontent.com/3851540/96956256-3cee6c00-152a-11eb-9499-e785a22014bc.png)

        ```bash
        dnf module info redis:remi-6.0/common
        ```

        ![4withprofile](https://user-images.githubusercontent.com/3851540/96956258-3d870280-152a-11eb-989a-8ab1a4e55842.png)

4. 啟用 module stream

    - 範例

        ```bash
        dnf module enable -y {package name}:{stream name}
        ```

    - 語法

        ```bash
        dnf module enable -y redis:remi-6.0
        ```

        ![5enablestream](https://user-images.githubusercontent.com/3851540/96956259-3d870280-152a-11eb-9138-0e3596ceb531.png)

5. 指定舊版的 artifact 安裝套件

    > 可以安裝非最新版的套件

    - 語法

        ```bash
        dnf install -y --enablerepo={repo name} {artifact name}
        ```

    - 範例

        ```bash
        dnf install -y --enablerepo=remi redis-0:6.0.7-1.el8.remi.x86_64
        ```

        ![6installredis](https://user-images.githubusercontent.com/3851540/96956260-3e1f9900-152a-11eb-995a-f6e0273d88a6.png)

## 心得

想不到 dnf 多了個 module 讓安裝不同版本變得複雜不少XD，不過這是我個人感受，我找了好幾個小時的資料都沒找到明確的解決方式，不知道是我關鍵字下得不好、我太不了解 dnf 或是只有我有這個需求

不論如何，就快速紀錄一下，我相信這樣的流程跟語法我覺得我不會一直都記得@@"

## 參考資訊

1. [yum 安裝指定版本套件](/yum-specific-version/)
2. [使用 dnf 安裝 Redis 6](/dnf-install-redis6/)
3. [CentOS 7/8 yum安装指定版本Redis](https://blog.csdn.net/coco3848/article/details/107605687)
