---
title: "Git 修改資料夾名稱的做法"
date: 2017-05-26T01:00:00+08:00
lastmod: 2021-11-02T01:00:05+08:00
draft: false
tags: ["Git"]
slug: "git-mv"
aliases:
    - /2017/05/git-mv.html
---
## Git 修改資料夾名稱的做法

同事想要調整 Git repository 的 folder 名稱，以往 SVN 的修改方式就是直接修改資料夾名稱，然後 commit，一切搞定：因為 SVN 是檔案 base 的版控，它可以持續追蹤到修改資料夾名稱，其他一切如故，但 Git 的運作方式不同，這樣的操作對於 Git 就會變成移除檔案再新增檔案的情境，檔案還是會在，但對於 Git 來說檔案已經不是同個檔案了，變成 rename 前的資料夾內容都變成 `Missing`，rename 後的資料夾內容都是 `Not Versioned`，當然可以手動刪除 `Missing` 的檔案及狀態，但這樣的操作既麻煩又沒有意義，我們來看看 Git 中的正確做法

## 基本情境設定

1. 在 folder 中建立 `test` 資料夾
2. 在 `test` 建立 `1.txt`
3. 依序在 `1.txt` 中建立幾個版本
4. 將 `test` 改名為 `test123`

## 直接修改資料夾名稱

1. 直接在檔案總管中修改資料夾名稱
2. Git 會將原本資料夾內容標記會 `Missing` 並將 rename 完的資料夾標記為 `Not Versioned`

    ![1renamefolder](https://cloud.githubusercontent.com/assets/3851540/26442618/df9f5cf6-4167-11e7-91f8-b44cf2ca2e42.png)

## 透過 Git 修改

1. 使用指令
    * 指令範例(詳細內容可以參考 [git-mv](https://git-scm.com/docs/git-mv))

        > `git mv test test123`

    * 使用 git status 檢查狀況 --> 正確標記為 `renamed`

        ![3gitstatus](https://cloud.githubusercontent.com/assets/3851540/26442621/dfc488dc-4167-11e7-85f8-b6076ea3d193.png)

2. 使用 `TortoiseGit`

    * 資料夾--> 按右鍵 --> TprtoiseGit --> Rename..

        ![4rightclivk](https://cloud.githubusercontent.com/assets/3851540/26442623/dfc731b8-4167-11e7-8538-db2b596083c1.png)

    * New name

        ![5newname](https://cloud.githubusercontent.com/assets/3851540/26442620/dfc3e08a-4167-11e7-99d4-c8be513dca80.png)

    * 正確標記為 `Rename`

        ![6rename](https://cloud.githubusercontent.com/assets/3851540/26442617/df9e1bca-4167-11e7-81c7-0d88ee569036.png)

## 關於檔案的修改歷程

* 不管哪一種修改方式，檔案的修改歷程都會被重設，舊的歷程不會再以個別 commit 顯示(無法使用 diff with previous version)

  * 原檔案歷程

        ![7commithistroy](https://cloud.githubusercontent.com/assets/3851540/26442614/df9bc87a-4167-11e7-95d4-c455c276796e.png)

  * 新檔案歷程

        ![2filehistory](https://cloud.githubusercontent.com/assets/3851540/26442622/dfc57bca-4167-11e7-9a87-6092b4279ecc.png)

* 可以使用 git blame 看修改歷程

  * 檔案 --> 按右鍵 --> TortoiseGit --> Blame

        ![10blame](https://cloud.githubusercontent.com/assets/3851540/26442619/dfa0c0a0-4167-11e7-9dc1-12f5e5f3cdb2.png)

* 可以使用指令可修改歷程

  * 指令 1

    * 語法

            ```
            git log --follow {file}
            ```

    * 範例

            ```
            git log --follow 1.txt
            ```

        ![9gitlog](https://cloud.githubusercontent.com/assets/3851540/26442616/df9ddd54-4167-11e7-939d-3b49e3ab5469.png)

  * 指令 2

    * 語法

            ```
            gitk --follow {file}
            ```
    * 範例

            ```
            gitk --follow 1.txt
            ```

        ![8gitk](https://cloud.githubusercontent.com/assets/3851540/26442615/df9bc7da-4167-11e7-945f-941ba9c64a1b.png)

## 參考資訊

1. [git-mv](https://git-scm.com/docs/git-mv)
2. [View the change history of a file using Git versioning](https://stackoverflow.com/questions/278192/view-the-change-history-of-a-file-using-git-versioning)
3. [Preserving History When Renaming Files in git](http://thisbythem.com/blog/preserving-history-when-renaming-files-in-git/)
