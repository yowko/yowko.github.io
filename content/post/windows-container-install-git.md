---
title: "為 Windows Container 安裝 Git - Part2"
date: 2019-02-19T20:40:00+08:00
lastmod: 2019-02-19T20:40:30+08:00
draft: false
tags: ["Docker","Windows","Git"]
slug: "windows-container-install-git"
---
# 為 Windows Container 安裝 Git - Part2

昨天提到同事想要在 Windows Container 上安裝 Git 當做 base image 再用來擴充其他功能，但沒辦法直接安裝，所以在 [為 Windows Container 安裝 Git](https://blog.yowko.com/windows-container-git) 介紹到透過 Chocolatey 安裝 Git 的做法

雖然知道解法不是很漂亮，但還是有解決問題，想不到同事嫌得要死覺得不該裝多餘的東西，所以我又來了@@

## 基本環境說明

1. Windows 10 Version 1803 (OS Build 17134.590)
2. Docker Community 18.09.2 (windows/amd64)

## Git for Windows 參數

> Git for Windows 已支援可以進行 silent install

1. 列出可用參數

    ```
    Git-2.20.1-64-bit.exe /?
    ```
2. 詳細說明

    ![1parameters](https://user-images.githubusercontent.com/3851540/53022599-7558bf80-3496-11e9-88a4-0363d4e6b66d.png)

    > 可使用 `/SILENT` 或是 `/VERYSILENT` 參數來進行安裝

## Dockerfile

```
FROM mcr.microsoft.com/windows/servercore
#將 git 安裝檔置於 `setup` 資料夾中，再將資料夾 mount 至 container 的 c:/setup 資料夾
ADD ./setup c:/setup
# 執行 git 安裝檔並使用 `/VERYSILENT` 參數進行安裝
RUN ["c:/setup/Git-2.20.1-64-bit.exe","/VERYSILENT"]
```

## 建置與實際使用
1. 先下載 [Git for Windows](https://git-scm.com/download/win) 並存放在 dockerfile 所在位置的 setup 資料中

    ![2setupfolder](https://user-images.githubusercontent.com/3851540/53022597-7558bf80-3496-11e9-867e-430b1b909bc5.png)

2. 建置 image

    - 格式
    
        ```cmd
        docker build {dockerfile folder} -t {repository:tag}
        ```
    
    - 範例
        ```cmd
        docker build c:\WinContainerGit -t windowswithgit:v2
        ```
        
        > `c:\WinContainerGit` 為放 `dockerfile` 檔案的資料夾

    ![3buildimage](https://user-images.githubusercontent.com/3851540/53022598-7558bf80-3496-11e9-9baf-a1f2912cf1a5.png)

3. 建立 container 並確認成功安裝 Git

    ```cmd
    docker run -it windowswithgit:v2 git --version
    ```

    ![1gitversion](https://user-images.githubusercontent.com/3851540/52961640-601c5c00-33d6-11e9-9edc-dfb6abd45704.png)

## 心得
同事真的是互相漏氣求進步呀，嘴巴很賤卻是督促自己精益求精的原動力，要不是今天被同事嘴了幾句，我想我應該會把印象停留在只能透過 Chocolatey 安裝 Git 

很感謝同事的激勵，讓我可以導正做法，不用再做多餘的安裝.......

# 參考資訊
1. [為 Windows Container 安裝 Git](https://blog.yowko.com/windows-container-git)
2. [Git for Windows](https://git-scm.com/download/win)
3. [Git For Windows Silent Install Silent Arguments](https://superuser.com/a/1112382)