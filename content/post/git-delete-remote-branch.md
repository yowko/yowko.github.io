---
title: "關於 Git 刪除 Remote Branch"
date: 2017-08-03T23:56:00+08:00
lastmod: 2018-09-24T23:56:32+08:00
draft: false
tags: ["Git"]
slug: "git-delete-remote-branch"
aliases:
    - /2017/08/git-delete-remote-branch.html
---
# 關於 Git 刪除 Remote Branch
今天同事問到為什麼從 Git Server 上刪除 branch 後，local 還是看得到被刪除的 branch，仔細想想我好像沒有這樣操作過，所以做了幾個實驗，提供參考

## 問題描述

1.  Git Server 上有兩個 branch：`develop` 與 `master`

2.  刪除 develop 後，local 還是看得到 `remotes/origin/develop`

    ![1deleted](https://user-images.githubusercontent.com/3851540/28930838-c3a3406e-78a6-11e7-84a0-3752dc507161.png)

    ![2local](https://user-images.githubusercontent.com/3851540/28930840-c3ca0c1c-78a6-11e7-9fbf-be2288f59dc1.png)

3.  使用 fetch 指令，狀況依舊

## 解決方式

1.  由 local 刪除遠端 branch

    > `git push origin :develop`

    *   當前目錄的 `remotes/origin/develop` 就會一併被刪除

        ![3localdelete](https://user-images.githubusercontent.com/3851540/28930959-22a0f8a4-78a7-11e7-9493-a3207ead804a.png)

    *   其他目錄或是電腦仍然有相同問題

2.  使用 fetch 指令進行修正

    *   2-1. git fetch -p

        ![5fetchp](https://user-images.githubusercontent.com/3851540/28930842-c3e783fa-78a6-11e7-9627-edf8c209c9c4.png)

    *   2-2. git fetch --all --prune

        ![4prunall](https://user-images.githubusercontent.com/3851540/28930843-c3eb9cd8-78a6-11e7-9ad2-7ed72477fe14.png)

    *   參數說明


        *   `--all`

            > 針對所有的 remote 進行 fetch

        *   `-p` or `--prune`

            > 執行 fetch 前，將遠端不存在的參考都移除

## 心得

Git 功能好多呀，指令也好多，最近同事常常會問出我從來沒想過的情境，讓我多了不少思考跟找指令的機會，也讓我聯想到其實我似乎沒有寫程式的天份，我怎麼都不會想到同事們會想到的情境，難道我該轉換跑道了嗎 XD

# 參考資訊

1.  [git-fetch](https://git-scm.com/docs/git-fetch)
2.  [fetch from origin with deleted remote branches?](https://stackoverflow.com/questions/5751582/fetch-from-origin-with-deleted-remote-branches)
3.  [How do I delete a Git branch both locally and remotely?](https://stackoverflow.com/questions/2003505/how-do-i-delete-a-git-branch-both-locally-and-remotely)
