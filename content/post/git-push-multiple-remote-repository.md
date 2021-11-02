---
title: "Git 如何設定一次 Push 至多個 Remote Repository"
date: 2017-04-28T01:00:00+08:00
lastmod: 2021-11-02T20:00:41+08:00
draft: false
tags: ["Git"]
slug: "git-push-multiple-remote-repository"
aliases:
    - /2017/04/git-push-multiple-remote-repository.html
---
## Git 如何設定一次 Push 至多個 Remote Repository

相信你一定不想看到辛苦寫完的程式，因為意外造成 Source Code 遺失，當然還擇良好的 Git server 服務是必要條件，但再穩固的服務也有可能出現被 DDOS 攻擊或是服務管理員不小心把 DB 刪除...等等，各式各樣你意想不到的情境造成服務中斷，所以為 remote repository 進行備份也是應該的

雖然備份很重要，但身為一個 ~~偷懶~~ 講求效率的工程師，你一定也不想每次 push 到 remote 都要反覆操作 push 多次，就我們來看看可以如何設定，讓一次動作就可以同時 push 到多個 remote repository

## 設定 remote url

> ~~這個設定動作只能使用 `指令` 執行~~

* 原本專案的 remote 設定

    ![1defaultremote](https://cloud.githubusercontent.com/assets/3851540/25490928/f7038070-2ba0-11e7-8558-5b5ba7b19faa.png)

* 使用指令加入其他 remote

  * HTTPS

        > `git remote set-url --add --push origin https://gitserver/repository.git`

    * 範例

            > `git remote set-url --add --push origin https://github.com/yowko/TestMilestone.git`

  * SSH

        > `git remote set-url --add --push origin ssh://git@gitserver:username/application.git/`

    * 範例

            > `git remote set-url --add --push origin git@github.com:yowko/TestMilestone.git`

* 注意事項

  * 如果原本已有 remote 設定，需要再手動 add 一次，否則僅會覆蓋 push 的設定

        ![2resetsetting](https://cloud.githubusercontent.com/assets/3851540/25490926/f702c9c8-2ba0-11e7-818a-b67590f8f929.png)

## 一次 push 至多個 remote

* 確認是否已設定多個 remote

  * `git remote -v`

        ![3multipleremote](https://cloud.githubusercontent.com/assets/3851540/25490927/f7036c2a-2ba0-11e7-83e0-c1829cc3dc24.png)

* push 至 remote

  * `git push origin`

        ![5success](https://cloud.githubusercontent.com/assets/3851540/25490923/f6fee448-2ba0-11e7-90e9-aca63d1240d3.png)

  * gui 工具

        ![4multiplesuccess](https://cloud.githubusercontent.com/assets/3851540/25490924/f7007c2c-2ba0-11e7-89af-7bc50d68e352.png)

## 參考資訊

1. [Git push to multiple remotes](http://blog.deadlypenguin.com/blog/2016/05/02/git-push-multiple-remotes/)
2. [Git - Pushing code to two remotes](http://stackoverflow.com/questions/14290113/git-pushing-code-to-two-remotes)
