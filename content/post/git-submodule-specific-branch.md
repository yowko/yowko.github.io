---
title: "Git Submodule 指定 Branch"
date: 2017-10-12T23:25:00+08:00
lastmod: 2021-11-02T23:25:36+08:00
draft: false
tags: ["Git"]
slug: "git-submodule-specific-branch"
aliases:
    - /2017/10/git-submodule-specific-branch.html
---
## Git Submodule 指定 Branch

曾經在 [Git 專案引用其他 Repository 的作法(Git Submodule)](/git-submodule) 介紹過如何使用 Git submodule 來引用其他 repository 內容，最近同事在使用 Git submodule 時發現預設使用 master branch，問到該如何指定非預設的 master branch，順手紀錄一下

## 使用特定 branch 加入 submodule

1. 使用指令
    * 語法

        ```cmd
        git.exe submodule add -b {BranchnName} -- "{GitRepositoryURL}" "{FolderName}"
        ```

    * 指令範例

        ```cmd
        git.exe submodule add -b NewBranch -- "http://github.com/yowko/TestSub-sub.git" "submodule"
        ```

2. 使用 TortoiseGit
    * 在欲加入其他 repository 的資料夾中按右鍵 --> TortoiseGit --> Submodule Add..

        ![1subadd](https://user-images.githubusercontent.com/3851540/31503797-2da2cc72-afa3-11e7-99ed-caa6255ae275.png)

    * 填入 `Repository Url`,`Folder Name`,`Branch Name`

        ![2urlbranch](https://user-images.githubusercontent.com/3851540/31503798-2dccd5da-afa3-11e7-90c4-37c9cf4d380c.png)

## 參考資訊

1. [Git Submodule 用法筆記](http://blog.chh.tw/posts/git-submodule/)
2. [7.11 Git Tools - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
3. [git-submodule last updated in 2.14.2](https://git-scm.com/docs/git-submodule)
