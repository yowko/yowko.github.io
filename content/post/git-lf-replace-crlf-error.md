---
title: "使用 Git ADD 指令出現錯誤"
date: 2017-11-17T23:54:00+08:00
lastmod: 2021-11-02T23:54:52+08:00
draft: false
tags: ["Git"]
slug: "git-lf-replace-crlf-error"
aliases:
    - /2017/11/git-lf-replace-crlf-error.html
---
## 使用 Git ADD 指令出現錯誤

同事反應無法使用 TortoiseGit 將新檔案 commit，後來進一步發現連加入 index 都不行，我接手後發現原現 `git add` 指令無法正常使用，一度以為是安裝 Git 時 CRLF 轉換的設定選錯，還嘗試重新安裝，但問題一樣存在，最後眼角餘光瞄到一個設定怪怪的，總算解決問題，立馬筆記一下

## 錯誤訊息

1. 訊息內容

    ```log
    fatal: LF would be replaced by CRLF in c3.css
    ```

    ```log
    Error: libgit2 returned: LF would be replaced by CRLF in 'c3.css'
    ```

2. 錯誤截圖

    ![1error1](https://user-images.githubusercontent.com/3851540/32955747-d3704c5a-cbf1-11e7-8113-b2745989218a.png)

    ![2error2](https://user-images.githubusercontent.com/3851540/32955748-d39c8b58-cbf1-11e7-9545-08e98afbb9cd.png)

## 解決方式

1. 取消 LF 與 CRLF 的轉換

    >LF 與 CRLF 的轉換是源自，Windows (使用 CRLF)與 UNIX (使用 LF) 兩個平台的換行符號不同造成的，而 GIT 源自 UNIX ，所以預設是使用 LF，所以在 Windows 上就會需要進行轉換(checkout 時將 LF 轉為 CRLF，commit 時則過來)

    * Windows 安裝 Git 時即預設轉換

        ![3default](https://user-images.githubusercontent.com/3851540/32955749-d3d0756c-cbf1-11e7-99f9-52e6f675b550.png)

    * 手動指令取消轉換

        ```cmd
        git config --global core.autocrlf false
        ```

    * 調整 `.gitconfig`

        > 檔案位於 `"C:\Users\{username}\.gitconfig"`

        ```config
        [core]
        autocrlf = false
        ```

        ![4config](https://user-images.githubusercontent.com/3851540/32955751-d4007988-cbf1-11e7-81f2-4f6116b22081.png)

    * 可能問題

        > Windows 原生的檔案格式就會跑掉

2. 檢查 SafeCRLF 設定

    > 同事設定取消 LF 轉換還是一樣無法 add 檔案，經檢查才發現是誤設定 `SafeCRLF`，移除後就正常了

    * 錯誤設定

        ![5safecrlf](https://user-images.githubusercontent.com/3851540/32955752-d42dd7b6-cbf1-11e7-9ec3-bf658ab4a260.png)

    * 成功 add

        ![6ok1](https://user-images.githubusercontent.com/3851540/32955753-d45eec3e-cbf1-11e7-8dde-8d2f23a93e1d.png)

        ![7ok2](https://user-images.githubusercontent.com/3851540/32955754-d48a200c-cbf1-11e7-8b1d-3b7df2c7a237.png)

## 心得

我第一時間並不曉得 `SafeCRLF` 的用途，經過查詢才清楚用途：

* true

    > 不允許 add 有 `LR` 與 `CRLF` 混合的檔案

* false

    > 允許 add 有 `LR` 與 `CRLF` 混合的檔案

* warn

    > 允許 add 有 `LR` 與 `CRLF` 混合的檔案，但會出 warning

大膽猜測同事應該不是刻意設定 `SafeCRLF`，可能在某些操作時不小心設定到的，實在是神手呀，用那麼久第一次看到這個設定，但也因為同事神手才讓我多暸解一個設定的含意，不過好像幫助有限XD

## 參考資訊

1. [Git中的AutoCRLF與SafeCRLF換行符問題](http://www.cnblogs.com/flying_bat/p/3324769.html)
