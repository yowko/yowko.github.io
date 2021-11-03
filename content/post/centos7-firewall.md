---
title: "CentOS 7 設定防火牆允許特定 PORT 連線"
date: 2017-09-29T00:08:00+08:00
lastmod: 2021-11-03T00:08:30+08:00
draft: false
tags: ["Linux"]
slug: "centos7-firewall"
aliases:
    - /2017/09/centos-7-firewall.html
    - /2017/09/centos7-firewall
---
## CentOS 7 設定防火牆允許特定 PORT 連線

一直以來接觸的作業系統都是 Windows 為主，加上自己熟悉的程式語言是 C#，production 環境部署及相關設定都是 Windows，也就一直沒有機會熟悉 Linux ，但軟體界有愈來愈開放的趨勢，微軟更是如此，現在微軟甚至是 GitHub 上最大的貢獻者

也因為 open source 的盛行，讓很多方便好用的工具跟軟體漸漸多元了起來，讓軟體開發也更加有趣，只是常見的 open source 工具多數以 linux 為主，就算有 porting 至 windows ，穩定性及更新速度上還是遠不及 linux 版本，以 redis 為例，linux 已經 release 4.0 版本，windows 還停留在一年前的 3.0 版本

今天遇到的問題就是因為 redis 而起的，因為公司 redis 架設在 linux 上，為了模擬 production issue，所以透過 virtualbox 安裝 CentOS 7 並在上面架設 redis instance，在測試連線時發現根本無法對外服務，就來看看如何確認問題及解決吧

## 問題確認

1. 無法連線 redis instance

    ![1connectdeny](https://user-images.githubusercontent.com/3851540/30976026-5aa56110-a4a6-11e7-976c-47d8f0f5c451.png)

2. 確認防火牆是否開啟

    ```bash
    firewall-cmd --zone=public --list-all
    ```

    ![2list](https://user-images.githubusercontent.com/3851540/30976029-5b1a3206-a4a6-11e7-818f-3e6491b61dc1.png)

## 解決方式

1. 對外開放 6379 port

    ```bash
    firewall-cmd --zone=public --add-port=6379/tcp --permanent
    ```

    > `--permanent` 指定為永久設定，否則在 firewalld 重啟或是重新讀取設定，就會失效

2. 重新讀取 firewall 設定

    ```bash
    firewall-cmd --reload
    ```

3. 可以檢查是否成功加入開放清單

    ```bash
    firewall-cmd --zone=public --list-all
    ```

    ![3addoirt](https://user-images.githubusercontent.com/3851540/30976027-5aa5b250-a4a6-11e7-977c-ae6f985a9eab.png)

## 設定成功

![4success](https://user-images.githubusercontent.com/3851540/30976028-5ab1a880-a4a6-11e7-92ba-0a4e4cf1a29b.png)

## 心得

上述動作非常簡單，但我卻找了好一陣子才真正解決問題，主因就是對於作業系統，我一直都只會操作 Windows，不確定上述的作法是否合乎 linux 的操作模式，也不清楚這樣設定會不會造成安全疑慮，只是剛好遇到問題紀錄一下，如果各位朋友有更好的做法還煩請指教，感激不盡

## 參考資訊

1. [How to open a port in the firewall on CentOS or RHEL](http://ask.xmodulo.com/open-port-firewall-centos-rhel.html)
2. [Centos7 - 新的防火牆firewalld](http://n.sfs.tw/content/index/10384)
