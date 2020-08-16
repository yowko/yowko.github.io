---
title: "CentOS 使用 yum 安裝 Redis6"
date: 2020-08-16T14:30:00+08:00
lastmod: 2020-08-16T21:30:31+08:00
draft: false
tags: ["Redis","Linux"]
slug: "centos-yum-redis6"
---
## CentOS 使用 yum 安裝 Redis6

之前筆記 [使用 Yum 安裝 Redis 5](https://blog.yowko.com/yum-install-redis5/) 中提到預設的 centos yum repository 中僅能安裝 redis3，透過使用 ius repository 才能使用 yum 來安裝 redis5，不過 redis6 已經問市一陣子，但 ius repository 尚未推出對應的更新，為了能使用 yum 來安裝 redis6，找了其他 yum repository，簡單快速紀錄一下

## 基本環境說明

1. Azure VM B2s (2 vcpu,4GiB memory)
2. OpenLogic 7.7
3. Redis 6.0.6

## 安裝方式

1. 下載 rpm

    ```bash
    curl -L -O http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
    ```

    或是

    ```bash
    wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
    ```

2. 安裝 remi rpm

    ```bash
    rpm -Uvh remi-release-7.rpm
    ```

3. 安裝 redis6

    > 透過 `yum info redis` 發現 `redis` 這個 package name 已是 redis6.0.6 (如果想要安裝其他版本，可以參考之前筆記 [yum 安裝指定版本套件](https://blog.yowko.com/yum-specific-version))

    ![1yuminfo](https://user-images.githubusercontent.com/3851540/90330536-03582c80-dfe0-11ea-84a4-cc75c7899fea.png)

    ```bash
    yum install -y redis
    ```

4. 驗明正身

    ```bash
    redis-server --version
    ```

    ![2redisversion](https://user-images.githubusercontent.com/3851540/90330539-05ba8680-dfe0-11ea-8e37-8316d4f30600.png)

## 心得

同事問到為什麼要堅持用 yum 安裝相關服務，明明可以自己從 source code build 起？ 我個人的想法：

1. 步驟較少也較單純

   > 我只想安裝相關功能，不想另外花心思前置安裝中繼編譯的套件與下載原始碼、後續刪除中繼套件跟原始碼，步驟愈多就愈容易出錯也愈不容易維護

2. 執行時間較短

    > 這個原因跟第一點有關連，因為少了 build 的動作，會縮短一些時間

## 參考資訊

1. [使用 Yum 安裝 Redis 5](https://blog.yowko.com/yum-install-redis5/)
2. [yum 安裝指定版本套件](https://blog.yowko.com/yum-specific-version)
3. [Install REMI Repository On RHEL, CentOS, Scientific Linux 7/6.x/5.x And Fedora 20/19/18](https://www.unixmen.com/install-remi-repository-rhel-centos-scientific-linux-76-x5-x-fedora-201918/)