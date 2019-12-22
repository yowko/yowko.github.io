---
title: "使用 Homebrew 安裝舊版 Helm"
date: 2019-12-21T21:30:00+08:00
lastmod: 2019-12-21T21:30:31+08:00
draft: false
tags: ["macOS","Tools","Helm"]
slug: "brew-install-specific-version-helm"
---

## 使用 Homebrew 安裝舊版 Helm

前幾天筆記 [使用 Homebrew 安裝舊版工具](https://blog.yowko.com/brew-install-old-version/) 紀錄到使用 Homebrew 來安裝舊版本工具(筆記中以 mariadb 為例)，這幾天在 mac 上測試 Helm 功能時，需要配合團隊目前技術使用 Helm 2，所以希望讓 Homebrew 安裝 Helm 2 舊版本，而非當前版本 - Helm 3，原以為在有了 [使用 Homebrew 安裝舊版工具](https://blog.yowko.com/brew-install-old-version/) 技巧後應該可以快速解決問題，但卻卡關了，簡單紀錄一下問題以及解決方式備忘

## 基本環境說明

1. macOS Catalina 10.15.1
2. helm 2.16.1

## 操作步驟

1. 搜尋 `kubernetes-helm` 版本

    ```bash
    brew search kubernetes-helm@
    ```

2. 安裝 `kubernetes-helm 2.16.1`

    ```bash
    brew install kubernetes-helm 2.16.1
    ```

    ![1brewsearch](https://user-images.githubusercontent.com/3851540/71310201-5a573300-244c-11ea-8eb6-974f7c654dfd.png)

3. 手動將 helm 加至環境變數中

    ```bash
    export PATH="/usr/local/opt/helm@2/bin:$PATH"
    ```

## 錯誤訊息

1. 訊息內容

    ```txt
    Updating Homebrew...
    ==> Auto-updated Homebrew!
    Updated 1 tap (homebrew/cask-versions).
    No changes to formulae.

    Error: No available formula with the name "2.16.1"
    ==> Searching for a previously deleted formula (in the last month)...
    Warning: homebrew/core is shallow clone. To get complete history run:
    git -C "$(brew --repo homebrew/core)" fetch --unshallow

    Error: No previously deleted formula found.
    ==> Searching for similarly named formulae...
    Error: No similarly named formulae found.
    ==> Searching taps...
    ==> Searching taps on GitHub...
    Error: No formulae found in taps.
    ```

2. 錯誤截圖

    ![2installerror](https://user-images.githubusercontent.com/3851540/71310203-5a573300-244c-11ea-820b-436afc4f041e.png)

## 解決方式

1. 重新檢視 search 的結果與錯誤訊息

    > 會發現 brew 的提示訊息

    - search

        ```txt
        No formula or cask found for "kubernetes-helm@".
        ```

        ![3searchhint](https://user-images.githubusercontent.com/3851540/71310204-5aefc980-244c-11ea-9564-1cb721d61d37.png)

    - error

        ```txt
        Error: No available formula with the name "2.16.1"
        ```

        ![4errorhint](https://user-images.githubusercontent.com/3851540/71310205-5aefc980-244c-11ea-81c2-a30ee650af2c.png)

2. 嘗試從 GitHub 上找歷史紀錄

    >在 Homebrew 的 GitHub 中找 `kubernetes-helm` 的版本歷程 [https://github.com/Homebrew/homebrew-core/search?q=kubernetes-helm](https://github.com/Homebrew/homebrew-core/search?q=kubernetes-helm)，可以發現 `kubernetes-helm` 的搜尋結果很不同

    - kubernetes-helm

        > 只有一個 json 檔案，也沒有其他版本資訊

        ![5k8s-helm-hostory](https://user-images.githubusercontent.com/3851540/71310206-5aefc980-244c-11ea-8c1c-6e706e63a69d.png)

    - mariadb

        > 多數都有 .rb 檔案，且存在使用 brew search 的對應版本

        ![6marriadb-history](https://user-images.githubusercontent.com/3851540/71310207-5b886000-244c-11ea-83fe-02bb519d16c7.png)

3. fomula rename

    > 這個從 [https://github.com/Homebrew/homebrew-core/search?q=kubernetes-helm](https://github.com/Homebrew/homebrew-core/search?q=kubernetes-helm) 結果的 json 中可以發現 fomula 已被 rename：`kubernetes-helm` --> `helm`

    ![7rename](https://user-images.githubusercontent.com/3851540/71310208-5b886000-244c-11ea-8a79-36c0b2a858a7.png)

4. 重新使用 `helm` 安裝

    - brew search helm@

        ![8helmsearch](https://user-images.githubusercontent.com/3851540/71310209-5b886000-244c-11ea-93fd-f901f04745eb.png)

    - brew install helm@2

        ![9helminstall](https://user-images.githubusercontent.com/3851540/71310210-5b886000-244c-11ea-8036-e714f772030c.png)

## 心得

虧我安裝前還特地到 [Helm 官網](https://v2.helm.sh/docs/using_helm/#installing-helm) 確認安裝語法，就是怕直接搜尋會出現多個 fomula 而裝錯，結果問題反而是出在官網沒有即時更新說明文件XD  不過總歸一句還是我自己第一時間沒有仔細看 brew 的執行訊息讓我多花了不少時間 debug 

只是我自以為是地覺得畢竟過去不了解 homebrew 的用法，第一時間沒看出問題應該情有可原吧@@" 反正學習就這麼回事嘛：多踩些別人沒踩到的雷，也許下次就有機會比別人快解決問題囉

## 參考資訊

1. [使用 Homebrew 安裝舊版工具](https://blog.yowko.com/brew-install-old-version/)
2. [Quickstart Guide](https://v2.helm.sh/docs/using_helm/#installing-helm)
