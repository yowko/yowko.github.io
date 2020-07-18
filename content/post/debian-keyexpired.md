---
title: "執行 apt-get update 時遇到 GPG KEYEXPIRED"
date: 2020-07-18T16:30:00+08:00
lastmod: 2020-07-18T16:30:31+08:00
draft: false
tags: ["Linux","Docker"]
slug: "debian-keyexpired"
---

## 執行 apt-get update 時遇到 GPG KEYEXPIRED

這是在追查 kafka 問題時，使用 docker-compose 建立 local 環境時，在 kafka container 內透過 apt-get 安裝工具來調整 kafka 設定值時遇到的狀況

google 之後還試了好幾種方法才解決問題，順手紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker images

    - confluentinc/cp-kafka:5.5.0

        > Debian GNU/Linux 8 (jessie)

## 錯誤訊息

1. 訊息內容

    ```txt
    W: GPG error: http://archive.debian.org jessie Release: The following signatures were invalid: KEYEXPIRED 1587841717
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/87849408-faaa0300-c91a-11ea-986d-156f88459477.jpg)

## 解決方式

1. 不要 update 直接安裝 vim

    ```bash
    apt-get install -y vim --force-yes
    ```

2. 修改 `/etc/apt/sources.list`

    > `vi /etc/apt/sources.list`

    ```txt
    deb http://http.debian.net/debian jessie main contrib
    deb-src http://http.debian.net/debian jessie main contrib

    deb http://security.debian.org/ jessie/updates main contrib
    deb-src http://security.debian.org/ jessie/updates main contrib
    ```

3. 再次執行 `apt-get update`

    ```bash
    apt-get update
    ```

    ![2ok](https://user-images.githubusercontent.com/3851540/87849409-fc73c680-c91a-11ea-9fd1-322ce2cf3531.jpg)

## 心得

google 到最多人提及的方式是像 [Fix Ubuntu/Debian apt-get “KEYEXPIRED: The following signatures were invalid”](https://futurestud.io/tutorials/fix-ubuntu-debian-apt-get-keyexpired-the-following-signatures-were-invalid)：先找到 expired 的 key，然後針對這個 key 做 renew，但我試了之後  反而出現更多 KEYEXPIRED 的錯誤，但 expired 的 key 卻沒有變動，我不確定是不是我操作錯誤造成的

另外雖然最終解決了問題，但也突顯我其實不知道問題的來龍去脈，只能依 google 到的內容來嘗試，不僅浪費時間也不知道解法是不是正確的方向，但這個是我一直以來的痛，沒有找到適合的方式來避免，只能先紀錄下來，邊做邊學囉

## 參考資訊

1. [Debian 8 Jessie KEYEXPIRED 1587841717](https://unix.stackexchange.com/questions/598344/debian-8-jessie-keyexpired-1587841717/598420#598420)
2. [Fix Ubuntu/Debian apt-get “KEYEXPIRED: The following signatures were invalid”](https://futurestud.io/tutorials/fix-ubuntu-debian-apt-get-keyexpired-the-following-signatures-were-invalid)
