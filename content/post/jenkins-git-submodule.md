---
title: "在 Jenkins 2 使用有 Submodule 的 Git Repository"
date: 2017-05-17T23:30:00+08:00
lastmod: 2018-09-22T23:30:21+08:00
draft: false
tags: ["Git","Jenkins"]
slug: "jenkins-git-submodule"
aliases:
    - /2017/05/jenkins-2-git-repository-submodule.html
    - /2017/05/jenkins-git-submodule
---
# 在 Jenkins 2 使用有 Submodule 的 Git Repository
前幾天文章 [Git 專案引用其他 Repository 的作法(Git Submodule)](//blog.yowko.com/2017/05/git-submodule.html) 介紹了如何使用 Git Submodule 的功能來拆解專案的相依性，讓 repository 可以引用其他外部 repository 檔案或是 source code，已經滿足了一般開發的基本需求，但身為精益求精的工程師，CI - Continuous integration 的整合也是不可或缺的一環，就來看看 Jenkins 2 遇上使用 submodule 專案時該如何設定

## 安裝 Git plugin

*   安裝 `Git Plugin` [Git Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Git%20Plugin)
*   這個套件是使用 Git 的基礎，下面範例會使用這個套件 demo


## 建立新的 job

1.  Jenkins 首頁 --> New Item

    ![1newitem](https://cloud.githubusercontent.com/assets/3851540/26141256/80ebba8e-3b0d-11e7-9a26-2d7af4bd351a.png)

2.  Enter an item name --> Freestyle project

    ![2entername](https://cloud.githubusercontent.com/assets/3851540/26141257/80f10110-3b0d-11e7-84fc-9324956a2302.png)

## 設定 Source Code Management

1.  使用 Git

2.  填入 Repository URL

3.  選擇適合的 Credentials

    *   如果選錯 Credentials，會出現錯誤訊息

        ![3CREDERROR](https://cloud.githubusercontent.com/assets/3851540/26141258/80fdd80e-3b0d-11e7-8801-a45b6eafb178.png)

        ![4credpass](https://cloud.githubusercontent.com/assets/3851540/26141259/81067b44-3b0d-11e7-8c86-8df5ece959e4.png)

    *   可以透過 `ADD` --> `Jenkins` 新增 Credentials

        ![5addcred](https://cloud.githubusercontent.com/assets/3851540/26141251/80a5c36c-3b0d-11e7-963a-7daedf0fd0ef.png)

4.  增加 Additional Behaviours --> Advanced sub-modules behaviours

    ![6addbehaviours](https://cloud.githubusercontent.com/assets/3851540/26141252/80d2c718-3b0d-11e7-847d-237a30a31ffa.png)

5.  填入 submodule repository url

    > `Use credentials from default remote of parent repository`

    
    ![7moduleurl](https://cloud.githubusercontent.com/assets/3851540/26141253/80e2b7f4-3b0d-11e7-9b54-115d76dfca06.png)

6.  設定是否使用相同的 credential

    > 視情況勾選 `Use credentials from default remote of parent repository`

    ![8samecred](https://cloud.githubusercontent.com/assets/3851540/26141254/80e34336-3b0d-11e7-8f68-1bad9a990f42.png)

## 編譯結果

> 確認有執行 `git submodule init`、`git submodule update`，詳細請看 [Git 專案引用其他 Repository 的作法(Git Submodule)](//blog.yowko.com/2017/05/git-submodule.html)

![9result](https://cloud.githubusercontent.com/assets/3851540/26141255/80eadfb0-3b0d-11e7-9154-ada4e510193a.png)

# 參考資訊
1.  [Git 專案引用其他 Repository 的作法(Git Submodule)](//blog.yowko.com/2017/05/git-submodule.html)
