---
title: "Git Force Push 後如何更新 Repository"
date: 2020-05-23T22:30:00+08:00
lastmod: 2020-12-11T22:30:31+08:00
draft: false
tags: ["Git"]
slug: "git-pull-after-force-push"
---

## Git Force Push 後如何更新 Repository

在之前筆記 [清除 Git Commit 紀錄](/clean-git-commit-history/) 提到打算將開發時期的 commit history 都清除，來個重新開始，但在完成 repository 整理後，團隊其他人還是得繼續往下開發，在找不到共同的 parent commit 下，`git pull` 無法正確執行，今天就來紀錄一下該如何不重新 clone 而取得 repository 的更新內容

## 基本環境說明

1. macOS Catalina 10.15.4
2. git version 2.23.0

## 錯誤訊息

1. 訊息內容

    ```txt
    From https://gitlab.com/yowko/xunit.net.testgenerator_test
      a1c4acd...1caf56a master     -> origin/master  (forced update)
      [new tag]         final      -> final
    fatal: refusing to merge unrelated histories
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/82734036-b51def00-9d4a-11ea-9ac0-355155a1dc4b.jpg)

## 解決方式

1. 直接使用新版：`reset`

    > 推薦使用，流程比較簡單直覺

    ```bash
    git reset origin/master --hard
    ```

2. 將 local 的變更重新加至新 master 上：`rebase`

    > 需要手動調整 conflict

    ```bash
    git rebase -i origin/master
    ```

## 心得

好一陣子沒有純粹使用 git cli 來操作，的確生疏不少，一部份原因是我個人無法捨棄 GUI 畫出來的 commit graph，雖然需要看的機會比想像的少得多，但最重要的主因還是不想記那麼多 git command，還要擔心下錯指令XD

## 參考資訊

1. [清除 Git Commit 紀錄](/clean-git-commit-history/)
2. [Git pull after forced update](https://stackoverflow.com/a/9813888)
