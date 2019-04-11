---
title: "如何在 Windows Server 2016 的 Hyper-V 上安裝 Centos 7"
date: 2017-05-03T01:00:00+08:00
lastmod: 2018-09-18T01:00:03+08:00
draft: false
tags: ["Linux","Windows Server 2016"]
slug: "windows-server-2016-hyperv-centos7"
aliases:
    - /2017/05/windows-server-2016-hyper-v-centos-7.html
---
# 如何在 Windows Server 2016 的 Hyper-V 上安裝 Centos 7
之前文章 [如何在 Windows Server 2016 上安裝 Hyper-V](//blog.yowko.com/2017/05/windows-server-2016-hyper-v.html) 提到為了想體驗 linux container 所以在 Windows Server 2016 上安裝了 Hyper-V 打算安裝 linux，Windows Server 2016 跟 Hyper-V 都已經準備就緒，接著就是來安裝 linux 了。

linux 版本很多，ubuntu、centos、debian、redhat....etc，我身為一個 linux 小白實在分不太清楚其中的差異，只大概知道 redhat 要錢、ubuntu 比較適合個人使用、debian 很精簡但適合熟手、centos 適合當 server，經過 ~~精心~~ 比較版本後就選 centos，而 centos 7 是最新版本，於是就這麼決定選 centos 7XD

## 建立虛擬交換器

> 建立 VM 用的網路功能

1.  Windows 系統管理工具 --> Hyper-V 管理員

    ![1hypervmanager](https://cloud.githubusercontent.com/assets/3851540/25579905/ae453728-2eae-11e7-9cd5-da0fcdd1625e.png)

2.  虛擬交換器管理員

    ![5virtualswitch](https://cloud.githubusercontent.com/assets/3851540/25579910/ae691b8e-2eae-11e7-83a4-e55fc3f4b9c9.png)

3.  虛擬交換器類型

    ![6publicswitch](https://cloud.githubusercontent.com/assets/3851540/25579911/ae6b0e8a-2eae-11e7-8227-c0ef1d21e0c0.png)

4.  虛擬交換器內容

    ![7switchcontent](https://cloud.githubusercontent.com/assets/3851540/25579912/ae71e66a-2eae-11e7-8f91-54db5c57dcf3.png)

## 新增 linux 虛擬機

1.  Windows 系統管理工具 --> Hyper-V 管理員

    ![1hypervmanager](https://cloud.githubusercontent.com/assets/3851540/25579905/ae453728-2eae-11e7-9cd5-da0fcdd1625e.png)

2.  新增虛擬機器

    ![2newvm](https://cloud.githubusercontent.com/assets/3851540/25579906/ae4b3790-2eae-11e7-836c-4bcbd2bfeeb0.png)

    *  在開始前

        ![3beforestart](https://cloud.githubusercontent.com/assets/3851540/25579907/ae67b078-2eae-11e7-8c49-cd1dcc01fd5f.png)

    *   指定名稱和位置

        ![4naming](https://cloud.githubusercontent.com/assets/3851540/25579909/ae692d18-2eae-11e7-9cbd-b5ac0798a49e.png)

    *   指定世代 --> 第一代

        ![3gen1](https://cloud.githubusercontent.com/assets/3851540/25579908/ae68b482-2eae-11e7-937f-396f92b3dde0.png)

    *   指派記憶體

        ![8memory](https://cloud.githubusercontent.com/assets/3851540/25579913/ae8b99b6-2eae-11e7-85ad-2daa913ea0ff.png)

3.  設定網路功能

    ![9network](https://cloud.githubusercontent.com/assets/3851540/25579914/ae8c6d8c-2eae-11e7-9c99-151127cee037.png)

4.  連結虛擬硬碟

    ![10virtualhd](https://cloud.githubusercontent.com/assets/3851540/25579915/ae8f683e-2eae-11e7-96cf-d675057d6c33.png)

    *   安裝選項 --> 從可開機 CD/DVD-ROM 安裝作業系統 --> 映像檔(.iso)

        ![11installos](https://cloud.githubusercontent.com/assets/3851540/25579916/ae9152fc-2eae-11e7-8979-7763f17cb643.png)

5.  摘要

    ![12summary](https://user-images.githubusercontent.com/3851540/45695370-da308680-bb93-11e8-9c1a-54d8e5b05232.png)

## 安裝 centos 7

1.  連線至 VM

    ![13connect](https://cloud.githubusercontent.com/assets/3851540/25579918/ae971cb4-2eae-11e7-9d3b-8fbcb3b224ba.png)

2.  啟動 VM

    ![14startup](https://cloud.githubusercontent.com/assets/3851540/25579919/aeb77b94-2eae-11e7-8618-c6bbda1e04b5.png)

3.  安裝檢查

    ![15check](https://cloud.githubusercontent.com/assets/3851540/25579901/ae41e262-2eae-11e7-8b4b-79120670c75e.png)

4.  進行安裝

    ![16lang](https://cloud.githubusercontent.com/assets/3851540/25579902/ae425bca-2eae-11e7-96e8-7af45232e432.png)

    ![17summary](https://cloud.githubusercontent.com/assets/3851540/25579903/ae4274d4-2eae-11e7-9c3a-a6d79d882bb3.png)

5.  成功安裝

    ![18success](https://cloud.githubusercontent.com/assets/3851540/25579904/ae42c290-2eae-11e7-8ef5-9e4e468334b0.png)

# 參考資訊
1.  [如何在 Windows Server 2016 上安裝 Hyper-V](//blog.yowko.com/2017/05/windows-server-2016-hyper-v.html)
2.  [Hyper-V安裝Ubuntu 16.04](http://blog.xuite.net/yh96301/blog/463474218-Hyper-V%E5%AE%89%E8%A3%9DUbuntu+16.04)
