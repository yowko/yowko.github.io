---
title: "使用 Yum 安裝 Redis 5"
date: 2020-02-08T10:30:00+08:00
lastmod: 2020-02-08T10:30:31+08:00
draft: false
tags: ["Linux","Redis","Tools"]
slug: "yum-install-redis5"
---

## 使用 Yum 安裝 Redis 5

最近嘗試撰寫 script 在 CentOS 上建立 Redis 環境，嘗試過程中無意間發現不同的 yum repository 間有不同的 redis 套件版本，像是常用的 `epel-release` 上的 redis 就是 `3.2.12`，雖說主要功能不受影響，但細微的設定或是新版本功能就無法使用，順手紀錄該如何使用 yum 在 CentOS 上安裝 Redis 5

## 基本環境說明

1. Azure VM B1s (1 vcpu,1GiB memory)
2. CentOS-based 7.7
3. Redis 5.0.7
4. epel-release.noarch 0:7-11
5. ius-release.noarch 0:2-1.el7.ius

## 安裝方式

- 預設的 yum repo

    - 透過 `yum repolist` 可以列出當前安裝的 yum repo 清單

        ![1defaultrepo](https://user-images.githubusercontent.com/3851540/74078486-4bf9c080-4a66-11ea-958a-3055ea1116ad.png)

    - 預設 repo 未包含 redis package，無法透過 yum 來安裝 redis

        ![2noredis](https://user-images.githubusercontent.com/3851540/74078488-4ef4b100-4a66-11ea-8343-45a303c4b0b9.png)

- 使用 epel repo

    - 安裝 epel repo

        > repo 安裝需要 `root` 權限，如果不是 `root` user，請加 `sudo`

        ```sudo
        yum install -y epel-release
        ```

    - epel 中 redis 的版本是 `3.2.12`

        > 找出 redis 的 package name

        ![3epelredis](https://user-images.githubusercontent.com/3851540/74078489-4f8d4780-4a66-11ea-9572-28b659b0e31b.png)

        > 使用 `yum info` 來檢視 package 詳細資訊

        ![4redisversion](https://user-images.githubusercontent.com/3851540/74078490-50be7480-4a66-11ea-831f-9a4075f55b7a.png)

- 使用 IUS repo

    - 安裝 IUS repo

        > 因 ius 相依於 epel repo ，如果沒安裝過要連帶安裝 epel-release package，以下提供兩種語法皆可正確完成安裝

        1. 社群版語法

            ```bash
            yum install -y  https://centos7.iuscommunity.org/ius-release.rpm
            ```

        2. 官網版語法

            ```bash
            yum install -y https://repo.ius.io/ius-release-el7.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
            ```

    - 多了 redis 4 與 5

        ![5redis45](https://user-images.githubusercontent.com/3851540/74078492-51570b00-4a66-11ea-810a-62468d310676.png)

        > redis 5 的版本是 `5.0.7`

        ![6redis5info](https://user-images.githubusercontent.com/3851540/74078493-51570b00-4a66-11ea-98a0-211228629854.png)

    - 安裝 Redis 5

        ```bash
        yum install -y redis5
        ```

    - 啟動 redis

        > service 的內容，在安裝當下已經建立，詳細內容可以看 `/usr/lib/systemd/system/redis.service`

        ```bash
        systemctl start redis
        ```

    - 確認版本

        ```bash
        redis-cli info server
        ```

        ![7redisversion](https://user-images.githubusercontent.com/3851540/74078494-51efa180-4a66-11ea-96db-c73a0ea64c84.png)

## 心得

反覆測試安裝 redis 的過程才留意到原來 epel repo 中的 redis 版本停留在 `3.2.12`，這才注意到 ius 的便利以及 ius 相依於 epel 的關係，原以為是沒什麼營養的指令紀錄，仔細端詳也是有不少學問的嘛，有刻意筆記果然是正確的決定

## 參考資訊

1. [IUS](https://ius.io/)
2. [IUS Repository for CentOS 7](https://plone.lucidsolutions.co.nz/linux/centos/ius-repository-for-centos-7)
