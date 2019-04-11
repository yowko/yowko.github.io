---
title: "TortoiseGit 與遠端操作的功能不見了？！"
date: 2017-11-27T23:29:00+08:00
lastmod: 2018-09-30T23:29:38+08:00
draft: false
tags: ["Git","Tools"]
slug: "tortoisegit-missing-options"
aliases:
    - /2017/11/tortoisegit-missing-options.html
---
# TortoiseGit 與遠端操作的功能不見了？！
同事反應他電腦上的 TortoiseGit 無法與遠端 Repository 互動(包括 push、fetch、pull...)，原本以為是權限問題，後來仔細一看發現 TortoiseGit 中所有與遠端互動的操作選項都不見，但 commit 功能是正常的。後來我透過指令針對 remote repository 操作則是正常，確認是 TortoiseGit 的問題

紀錄一下 TortoiseGit 選單的設定

## 找不到 push,fetch,pull

![1nopullpushfetch](https://user-images.githubusercontent.com/3851540/33274059-05df8982-d3ca-11e7-9895-cff856656e22.png)

## 確認問題

*   `SHIFT`+ 滑鼠右鍵 測試是否顯示選項

    ![2shift](https://user-images.githubusercontent.com/3851540/33274061-0611b5f6-d3ca-11e7-95d5-97cdc634f3da.png)

*   可以發現選單在使用 `SHIFT` + 滑鼠右鍵 後已正常出現


## 調整設定

*   滑鼠右鍵 --> TortoiseGit --> Settings

    ![3setting](https://user-images.githubusercontent.com/3851540/33274062-0648c9f6-d3ca-11e7-9c21-397fdb456908.png)

*   General --> Set Extend Menu Item

    *   勾選表示不顯示

        ![4donotshow](https://user-images.githubusercontent.com/3851540/33274063-0678a838-d3ca-11e7-9688-7a76893ddcc9.png)

    *   同事的設定

        ![5setextendmenu](https://user-images.githubusercontent.com/3851540/33274064-06a6ec16-d3ca-11e7-891b-94447f88e8e5.png)

*   取消勾選

    ![6unchecked](https://user-images.githubusercontent.com/3851540/33274065-06d85a94-d3ca-11e7-84a2-d416453c0ade.png)

*   重新登入 or 重新開機

    > 設定套用需要 user 登出後再重新登入才會生效

    ![7ok](https://user-images.githubusercontent.com/3851540/33274066-070bf728-d3ca-11e7-8aec-02a8af0e0d7e.png)

## 心得

同事是 Git 新手應該不是刻意自行設定次選單不顯示的項目，但他也想不起來為什麼造成這樣的結果，就點一點、選一選功能就都不見了XD

說到底也就是一個選項設定，只是我一開始並沒有發現原來 Set Extend Menu Item 是用來設定不顯示的，花了一些時間才找到問題原因，也才發現人家明明有很清楚的說明文字，這就是不看人家說明的下場 @@"

# 參考資訊

1.  [TortoiseGit context menu options disappeared](https://stackoverflow.com/questions/33304187/tortoisegit-context-menu-options-disappeared)
