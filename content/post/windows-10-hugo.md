---
title: "在 Windows 10 中建置 Hugo 開發環境"
date: 2018-10-18T02:40:00+08:00
lastmod: 2021-10-08T02:40:52+08:00
draft: false
tags: ["Tools","Hugo","Windows 10"]
slug: "windows-10-hugo"
---
## 在 Windows 10 中建置 Hugo 開發環境

之前考慮了好久，最後終於下定決心將個人筆記改用 Hugo 來建置，雖然一直都是使用 markdown 來寫筆記但過去將筆記放在 Blogger 上時必需要手動將 markdown 轉為 html 加上偷懶，markdown 在完成後就變得可有可無，也就沒有刻意保存跟更新，造成最後筆記內容就像是沒有 source code 的 production -- 相當失控呀，讓我花了比預期多上好幾倍的時間來逐一重新檢視及轉換過去的筆記

當然過程中相當崩潰也曾經考慮過去的就讓它都過去，從新開始，只是回想起動手寫筆記的初衷 - 為了幫助自己記憶與學習，如果因為轉換平台讓原本目的變得不再容易達成，對我來說反而是捨本逐未了

說到這表示我終於把過去筆記都轉完了，又可以開始紀錄最近遇到的狀況，首先從架設 Hugo 開發環境開始吧

## Why Hugo

Hugo 是由 Go 寫的靜態頁面網站產生器，用來將 markdown 文件轉換為 html 格式，其特點就是轉譯速度快，也因為如此在眾多方案中 (像是 `Jekyll`、`Hexo`、`Hugo`、`Pelican`等) 脫穎而出，另外官方的說明文件完整性也是一個重點，對於新手在入門時大幅降低了進入門檻

編譯速度比較部份請參考 [Why I switched from Octopress 2 to Hugo](https://conscientiousprogrammer.com/blog/2015/05/31/why-i-switched-from-octopress-2-to-hugo/)

## 安裝 Hugo

> 其他作業系統請參考 [Install Hugo](https://gohugo.io/getting-started/installing)

1. 安裝 Git

    ```cmd
    choco install git.install
    ```

2. 安裝 Go (1.11 以上版本)

    ```cmd
    choco install golang
    ```

3. 安裝 Hugo

    ```cmd
    choco install hugo
    ```

    > 過去安裝 Hugo 時需要手動將 Hugo 的執行路徑加入環境變數中，但近期版本似乎可以省掉這個步驟

4. 驗證完成安裝

    > 要能正常回傳版本資訊，不可以出錯誤訊息

    ```cmd
    hugo version
    ```

    ![1hugoversion](https://user-images.githubusercontent.com/3851540/47168075-95cf0c80-d332-11e8-9270-c4a4068fa664.png)

## 建立開發環境

1. 建立 hugo 站台

    - 語法

        ```cmd
        hugo new site {sitename}
        ```

    - 實際範例

        ```cmd
        hugo new site blog
        ```

    ![2hugonew](https://user-images.githubusercontent.com/3851540/47168076-95cf0c80-d332-11e8-84b9-4fb9c0f19f2a.png)
2. 將 hugo 站台加入 git 版控

    - 進入剛建立的 hugo 站台資料夾中

        ```cmd
        cd hugo
        ```

    - 將資料夾加入 git 版控

        ```cmd
        git init
        ```

    - 將 hugo 站台內容加至 staging area 中

        ```cmd
        git add .
        ```

    - commit

        ```cmd
        git commit -m "repo init"
        ```

3. 加入 hugo theme

    > 透過 submodule 方式加入 theme

    ```cmd
    git submodule add https://github.com/yowko/hugo-theme-even-more.git themes/even
    ```

    > 記得隨時 commit

4. 將 theme 中的 config.toml 複製至 root

    ```cmd
    copy themes\even\exampleSite\config.toml .
    ```

    ![3copy](https://user-images.githubusercontent.com/3851540/47168077-95cf0c80-d332-11e8-9483-6c104e47a495.png)

## 開始建立 markdown 文件

1. 在 root 資料夾下建立存放 markdown 的資料夾

    > 依 theme 不同，資料夾名稱也可能不同，請參考 theme 的設定

    ```cmd
    mkdir content\post
    ```

2. 建立 markdown 文件

    ```cmd
    hugo new post/yowkotest.md
    ```

    ![4newmd](https://user-images.githubusercontent.com/3851540/47168078-9667a300-d332-11e8-8094-51da0376fc8d.png)

## 測試

1. 啟動 Hugo 測試站台

    ```cmd
    hugo server /D
    ```

    ![5hugoserver](https://user-images.githubusercontent.com/3851540/47168079-9667a300-d332-11e8-8ab7-7367ca48f9f4.png)

    ![6website](https://user-images.githubusercontent.com/3851540/47168072-95367600-d332-11e8-8c6b-3253badd6ea2.png)

2. 測試 markdown 內容需將 draft 改為 `false`

    ![7result](https://user-images.githubusercontent.com/3851540/47168074-95cf0c80-d332-11e8-84b4-4552b9532441.png)

## 心得

雖然 hugo 文件滿齊全的，但我也花了不少功夫才真的建置成可用的 blog 站台，可能是一直被微軟的便利性保護地太好，遇到 open source 總是東卡西卡的，應該要想辦法克服，畢竟現在 open source 的資源很豐富，不僅數量又多品質又好呀，不過終於成功將個人筆記從 Blogger 轉移至 Hugo，真是開心呀

## 參考資訊

1. [Install Hugo](https://gohugo.io/getting-started/installing)
