---
title: "安裝 SQL Server 2017 出現 exit code 1638"
date: 2018-08-02T02:40:00+08:00
lastmod: 2021-10-15T02:40:52+08:00
draft: false
tags: ["DB","SQL Server"]
slug: "sql-server-1638"
aliases:
    - /2018/08/sql-server-1638/
---
## 安裝 SQL Server 2017 出現 exit code 1638

今天拿到新的工作用電腦，首先當然就是安裝所有開發必備工具與軟體，其中一個便是 local 開發用的 DB instance，既然都要重新安裝便選了最新的版本 MS - SQL Server 2017，想不到過去一鍵到底 順暢無比的安裝體驗都沒能複製到這次上，安裝了兩次都出現相同錯誤訊息

趁著安裝空檔，順便紀錄一下解決方式

## 錯誤訊息

1. 訊息內容

    ```txt
    The following error has occurred:

    VS Shell installation has failed with exit code 1638.

    For help, click: https://go.microsoft.com/fwlink?LinkID=20476&ProdName=Microsoft%20SQL%20Server&EvtSrc=setup.rll&EvtID=50000&ProdVer=14.0.2000.63&EvtType=0x5B39C8B9%25401434%25403
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/43579116-8e1e9582-9683-11e8-9696-34068d5fe39b.png)

## 解決方式

1. 移除 Microsoft Visual C++ 2017 Redistributable (x86) and (x64)
    - x86

        ![2x86](https://user-images.githubusercontent.com/3851540/43579117-8e60d6fe-9683-11e8-9d86-98f11467e9f4.png)
    - x64

        ![3x64](https://user-images.githubusercontent.com/3851540/43579118-8e8b8836-9683-11e8-86b3-2ff6187044b3.png)

2. 再次安裝 SQL Server
3. (如有需要)再次安裝 Microsoft Visual C++ 2017 Redistributable (x86) and (x64)
    - [x86 下載位置](https://go.microsoft.com/fwlink/?LinkId=746571)
    - [X64 下載位置](https://go.microsoft.com/fwlink/?LinkId=746572)

## 心得

同事說他安裝時沒遇到問題，查了一下文件似乎是與安裝順序有關，先安裝 SQL Server 2017 再安裝 Visual Studio 2017 就不會遇到問題，因為 Visual Studio 2017 安裝時需要 Microsoft Visual C++ 2017 Redistributable，但無論如何還是 bug 呀

雖然過去的安裝體驗很好，但也難保下次可以一樣順利，這也就是軟體開發迷人之處呀，你什麼時候都不知道下次會不會一樣正常運作  哈哈

## 參考資訊

1. [Help installing SQL Server 2017 - VS Shell installation has failed with exit code 1638](https://dba.stackexchange.com/questions/190090/help-installing-sql-server-2017-vs-shell-installation-has-failed-with-exit-cod)
