---
title: "使用 dnf 安裝 Redis 6"
date: 2020-10-21T21:30:00+08:00
lastmod: 2020-10-21T21:30:31+08:00
draft: false
tags: ["Linux","Redis"]
slug: "dnf-install-redis6"
---

## 使用 dnf 安裝 Redis 6

之前筆記 [使用 Yum 安裝 Redis 5](https://blog.yowko.com/yum-install-redis5/) 與 [CentOS 使用 yum 安裝 Redis6](https://blog.yowko.com/centos-yum-redis6/) 紀錄到在 CentOS 中分別使用不同 rpm 安裝 Redis 5 與 Redis 6 的做法，最近改用 dnf 做為套件管理工具，安裝方式有些不同，簡單做個筆記

## 基本環境說明

1. Azure VM Standard_B2s (CentOS-based 8.2 - Gen1)
2. Linux (centos 8.2.2004)
3. Linux Kernel 4.18.0-193.19.1.el8_2.x86_64
4. Redis 6.0.8

## 安裝步驟

1. 安裝 repo 前僅有 redis5

    ![1before](https://user-images.githubusercontent.com/3851540/96956538-f77e6e80-152a-11eb-8355-4c8e1b9d46e3.png)

2. 安裝 repo

    ```bash
    dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
    ```

3. 安裝 repo 後增加 remi-5.0 與 remi-6.0

    ![2after](https://user-images.githubusercontent.com/3851540/96956543-f9483200-152a-11eb-8c5f-e5cb08469585.png)

4. 使用 modular stream 來確認 redis 6 安裝版本資訊

    ```bash
    dnf module info redis:remi-6.0
    ```

    ![3dnfinfo](https://user-images.githubusercontent.com/3851540/96956547-fa795f00-152a-11eb-9572-411556759a7c.png)

5. 使用 modular stream 來安裝 redis 6

    > 預設安裝最新版本；如果需要安裝不同版本，請參考 [使用 dnf 透過 module 安裝指定版本套件](https://blog.yowko.com/dnf-module-install-specific-artifact)

    ```dnf
    dnf module install -y redis:remi-6.0
    ```

    ![4redisversion](https://user-images.githubusercontent.com/3851540/96956549-fbaa8c00-152a-11eb-8059-0f88c80316f7.png)

6. 啟動 service

    ```bash
    systemctl enable redis.service && systemctl start redis.service
    ```

## 心得

可以理解透過 module 來管理套件比較有條理，也許處理的速度上也可以比較快，但就目前以個人感覺來說跟之前拆分 repo 的做法很類似，過去只要找到正確的 repo rpm 就可以安裝，現在還需要找到對的 module，好像是又加上一層過濾的感覺，不過可能是我還沒充份了解的關係

## 參考資訊

1. [DNF Command Reference](https://dnf.readthedocs.io/en/latest/command_ref.html)
2. [Install and Configure Redis on CentOS 8](https://www.vultr.com/docs/install-and-configure-redis-on-centos-8)
3. [使用 Yum 安裝 Redis 5](https://blog.yowko.com/yum-install-redis5/)
4. [CentOS 使用 yum 安裝 Redis6](https://blog.yowko.com/centos-yum-redis6/)
5. [使用 dnf 透過 module 安裝指定版本套件](https://blog.yowko.com/dnf-module-install-specific-artifact)
