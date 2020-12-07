---
title: "安裝 mysql client"
date: 2020-12-07T21:30:00+08:00
lastmod: 2020-12-07T21:30:31+08:00
draft: false
tags: ["mysql","Linux"]
slug: "install-mysql-client-only"
---

## 安裝 mysql client

程式更迭的過程，隨著功能的增加，不免需要 db 的 change 來配合，為了增加安裝性，所以打算封鎖 user 登入 mysql server 操作，僅透過 mysql client 來進行 patch，但又不想完整安裝 mysql (server 與 client)，除此之外 我也會透過安裝 mysql client 在 container 中確認 mysql 連線狀況，為了方便查閱  快速紀錄一下

## 基本環境說明

1. CentOS 8.2.2004, Kernel 4.18.0
2. Debian 10, Kernel 5.4.49+

## 安裝語法

1. CentOS

    > 在 dnf 與 yum 中，`mysql` 代表 MySQL client 跟 shared libraries， server 則為 `mysql-server` 或是 `mysql-common`

    ![1infomysql](https://user-images.githubusercontent.com/3851540/101332816-c70f4b80-38b0-11eb-9879-a1433e605efb.png)

    - dnf

        ```bash
        dnf install -y mysql
        ```

    - yum

        ```bash
        yum install -y mysql
        ```

2. Debian

    - apt

        ```bash
        apt install -y default-mysql-client
        ```

    - apt-get

        ```bash
        apt-get install -y default-mysql-client
        ```

## 心得

原本看到 CentOS 都安裝 `mysql` 還覺得這不是我要的，仔細看內容加上使用 `dnf info mysql` 才發現原來是我不懂 XD

發現後立馬去跟同事道歉：前一天才跟同事講錯 QQ

## 參考資訊

1. [install mysql-client-5.7 on debian buster](https://superuser.com/a/1481211)
2. [CentOS Linux 5/6: Install MySQL Client Only](https://www.cyberciti.biz/faq/centos-linux-56-install-mysql-client-only/)
