---
title: "安裝 MongoDB client"
date: 2020-12-11T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["MongoDB","Linux"]
slug: "install-mongodb-client-only"
---

## 安裝 MongoDB client

之前筆記 [安裝 mysql client](/install-mysql-client-only) 紀錄到在 container 中安裝 mysql client 來進行連線測試，今天就來紀錄一下也是常用工具的 mongodb client

## 基本環境說明

1. CentOS 8.2.2004, Kernel 4.18.0
2. Debian 10, Kernel 5.4.49+
3. Debian 9, Kernel 5.4.39-linuxkit

## 安裝語法

1. CentOS

    - 加入 MongoDB 官方 repository

        ```bash
        echo '[mongodb-org-4.4]
        name=MongoDB Repository
        baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
        gpgcheck=1
        enabled=1
        gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc'| tee /etc/yum.repos.d/mongodb-org-4.4.repo
        ```

    - 安裝 MongoDB client

        ```bash
        dnf install -y mongodb-org-shell-4.4.2
        ```

2. Debian

    - 安裝必要套件

        ```bash
        apt update && apt install -y gnupg wget
        ```

    - 匯入 apt 用的 public key

        ```bash
        wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | apt-key add -
        ```

    - 設定 mongodb package source

        - Debian 9

            ```bash
            echo "deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/4.4 main" | tee /etc/apt/sources.list.d/mongodb-org.list
            ```

        - Debian 10

            ```bash
            echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main" | tee /etc/apt/sources.list.d/mongodb-org.list
            ```

    - 安裝 mongodb client

        ```bash
        apt update && apt install -y mongodb-org-shell
        ```

## 心得

不知道為什麼 mongodb 在 mongodb4 之後都需要額外指定 rpm 來源，所以在 container 內有時候會遇到權限問題

另外有嘗試下載 [MongoDB Shell](https://www.mongodb.com/try/download/shell)：雖然少了加 rpm source 的問題，但嘗試使用的過程中發現有兩個差異(後來放棄使用)

1. shell 名稱是 `mongosh` 與正常安裝的 `mongo` 不同 (問題不大 rename 後一樣能用)
2. `mongosh` 不支援 `--eval` 用法 (這個影響太大了，立馬決定放棄)

MongoDB 實在不得我心，明明文件算清楚，但照著做就是不一定可以用，寫筆記可能只差著順序或是 typo 問題，不寫急著用時不能動又很氣XD

## 參考資訊

1. [MongoDB: Install Client – Mongo Shell – Ubuntu, CentOS](https://www.shellhacks.com/mongodb-install-client-mongo-shell-ubuntu-centos/)
2. [Install MongoDB Community Edition on Red Hat or CentOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)
3. [Install ONLY mongo shell, not mongodb](https://stackoverflow.com/a/42001799)
4. [MongoDB Shell](https://www.mongodb.com/try/download/shell)
5. [Install MongoDB Community on Debian using .tgz Tarball](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian-tarball/)
6. [How To Install MongoDB 4.4 / 4.2 On Debian 9](https://www.itzgeek.com/how-tos/linux/debian/how-to-install-mongodb-on-debian-9-debian-8.html)
