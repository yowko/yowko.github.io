---
title: "使用 Homebrew 安裝 Docker"
date: 2019-07-20T21:30:00+08:00
lastmod: 2019-07-20T21:30:31+08:00
draft: false
tags: ["Mac","Docker"]
slug: "homebrew-docker"
---

## 使用 Homebrew 安裝 Docker

現在工作上都是使用 mac，但為了保留可以比較 macOS 與 Windows 在部份行為上的差異，自己的家用電腦一直都是 macbook pro 使用 bootcamp 開機直接進入 Windows 10，最近因為需要較多乾淨的環境進行測試，在私人電腦上使用 macOS 的時間也變長不少，今天為了要測試 container 相關特性，打算在 mac 安裝 docker for mac

Docker 官網的建議方式 [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/) 是透過下載 `Docker.dmg` 的安裝檔來執行安裝，但身為一個 ~~專案~~ 懶惰的工程師透過 command 安裝是既帥氣又專案的象徵，於是我來簡單筆記一下

## 基本環境說明

1. macOS Mojave 10.14.5
2. Homebrew 2.1.7

    - 安裝 Homebrew

        ```bash
        /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
        ```

    - 更新 `Homebrew`

        ```cs
        brew update
        ```

3. homebrew-cask (git version 6c2ad)

    - 安裝 `Homebrew-Cask`

        ```bash
        brew tap caskroom/cask
        ```

## 安裝 Docker

1. 使用 brew 指令安裝

    ```bash
    brew cask install docker
    ```

    [1installing](https://user-images.githubusercontent.com/3851540/61592532-11822080-ac07-11e9-9b34-91a9a23c92f7.png)

    [2installing](https://user-images.githubusercontent.com/3851540/61592533-11822080-ac07-11e9-9f38-c8e7444e3405.png)

2. 啟動 docker for mac

    ```bash
    open /Applications/Docker.app
    ```

    ![3start](https://user-images.githubusercontent.com/3851540/61592534-121ab700-ac07-11e9-8804-f5c530549116.png)

    ![4start](https://user-images.githubusercontent.com/3851540/61592535-121ab700-ac07-11e9-9e5a-f95ae571e2ae.png)

    ![5start](https://user-images.githubusercontent.com/3851540/61592536-121ab700-ac07-11e9-8b8a-7556b1b09683.png)

## 心得

其他 brew 相關指令

- 搜尋軟體

    ```bash
    brew cask search 軟體名稱
    ```

- 列出可以更新的軟體

    ```bash
    brew cask outdated
    ```

- 更新指定軟體

    ```bash
    brew cask install --force 軟體名稱
    ```

- 一次更新所有過時軟體

    ```bash
    brew cask install --force $(brew cask outdated | awk '{print $1}' | xargs)
    ```

- 移除指定軟體

    ```bash
    brew cask uninstall 軟體名稱
    ```

透過 homebrew 來安裝 docker 並不一定比較快，不過可以使用指令來執行安裝，就可以一併安裝其他軟體，不用自行逐一下載安裝，管理上較為便利

## 參考資訊

1. [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
2. [Homebrew](https://brew.sh/)
3. [「Mac」Homebrew：Mac 必裝的套件管理工具](https://diary.taskinghouse.com/posts/4766365-homebrew-essential-mac-suite-management-tools/)
