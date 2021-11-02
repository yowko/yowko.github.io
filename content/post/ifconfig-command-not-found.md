---
title: "Linux 找不到 ifconfig、wget 指令？！"
date: 2017-09-05T22:42:00+08:00
lastmod: 2021-11-02T22:42:34+08:00
draft: false
tags: ["Linux"]
slug: "ifconfig-command-not-found"
aliases:
    - /2017/09/ifconfig-command-not-found.html
---
## Linux 找不到 ifconfig、wget 指令？

公司部份服務架設在 Linux 環境上，讓之前完全沒有 Linux 操作經驗的我有些苦手，雖然不像 Windows 那麼習慣與熟悉，但學習不同的東西還是非常有趣的

今天剛好在模擬某個特定問題時，重新安裝了 CentOS 7，以往都是選擇網頁伺服器版本，今天心血來潮使用 `最小安裝`，馬上卡關XD 就來看看遇到的問題以及解決方式吧

## 錯誤訊息

* ifconfig
    1. 訊息內容

        ```bash
        [root@localhost ~]# ifconfig
        -bash: ifconfig：命令找不到
        ```

    2. 錯誤截圖

        ![1ifconfig](https://user-images.githubusercontent.com/3851540/30066500-afb30564-928a-11e7-8c4b-7cd57336cfe1.png)

* wget
    1. 訊息內容

        ```bash
        [root@localhost ~]# wget http://download.redis.io/releases/redis-4.0.1.tar.gz
        -bash: wget：命令找不到
        ```

    2. 錯誤截圖

        ![2wget](https://user-images.githubusercontent.com/3851540/30066506-b154f940-928a-11e7-9d43-cfe9069543e2.png)

## 解決方式

1. ifconfig
    * 改用 `ip addr`

        ![3ipaddr](https://user-images.githubusercontent.com/3851540/30066502-afd6f122-928a-11e7-9d3c-595d7c02dd65.png)

2. wget
    * 直接安裝 wget

        ![4wgetinstall](https://user-images.githubusercontent.com/3851540/30066505-b047d8a6-928a-11e7-8aae-d3e5292f0589.png)

## 該安裝哪個套件？

看到這是不是覺得奇怪？同樣是 command not found，為什麼解決問題的方法不同？！我看到 command not found 的第一念頭就是缺什麼裝什麼，但安裝 ifconfig 時就遇到問題：`No package ifconfig available.`

![7ifconfignotfound](https://user-images.githubusercontent.com/3851540/30066507-b24117a8-928a-11e7-9e12-4f0603cb0280.png)

表示 ifconfig 不是單獨存在的套件，而是附屬在其他套件下，這樣怎麼知道該裝哪個套件，此時就可以透過 `yum search` 來找到所屬套件名稱

1. ifconfig

    > `yum search ifconfig` 就可以發現 `ifconfig` 屬於 `net-tools` 的一部份

    ```bash
    [root@localhost ~]# yum search ifconfig
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
        * base: centos.cs.nctu.edu.tw
        * extras: centos.cs.nctu.edu.tw
        * updates: centos.cs.nctu.edu.tw
    =================================================================== Matched: ifconfig ====================================================================
    net-tools.x86_64 : Basic networking tools
    ```

    ![5searchifconfig](https://user-images.githubusercontent.com/3851540/30066503-afdf60a0-928a-11e7-879a-1ea33e4ec5e6.png)

2. wget

    > `yum search wget` 就可以發現 `wget` 就是套件本身了

    ```bash
    [root@localhost ~]# yum search wget
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
        * base: centos.cs.nctu.edu.tw
        * extras: centos.cs.nctu.edu.tw
        * updates: centos.cs.nctu.edu.tw
    =================================================================== N/S matched: wget ====================================================================
    wget.x86_64 : A utility for retrieving files using the HTTP or FTP protocols
    
    Name and summary matches only, use "search all" for everything.
    ```

    ![6searchwget](https://user-images.githubusercontent.com/3851540/30066504-afe749dc-928a-11e7-8f27-9ad22e3332a6.png)

## 心得

Linux 真的博大精深呀，好多東西觀念都跟 Windows 不同，實在是學不完呀，幸虧現在工作需要多了不少動力跟挑戰

## 參考資訊

1. [解決 ifconfig command not found](https://www.phpini.com/linux/fix-ifconfig-command-not-found)
2. [yum 用法整理](http://www.vixual.net/blog/archives/101)
3. [ifconfig 不見了 (CentOS 7.2 最小安裝)](http://shaurong.blogspot.tw/2016/02/ifconfig-centos-72.html)
