---
title: "ZeroBraneStudio 無法在 macOS Catalina 中開啟？！"
date: 2019-12-05T21:30:00+08:00
lastmod: 2019-12-05T21:30:31+08:00
draft: false
tags: ["Redis","Lua","macOS"]
slug: "zerobranestudio-macos-catalina"
---

## ZeroBraneStudio 無法在 macOS Catalina 中開啟？！

上個禮拜專案終於告個段落，立馬按下了幾個 pending 的更新，其中一個就是 macOS Catalina，基本上使用沒有發生異常，但許多軟體設定跑掉，但問題也不太大就是有些不順手，直到今天要 debug redis lua 時才發現 ZeroBraneStudio 完全無法開啟，筆記一下

## 基本環境說明

1. macOS Catalina 10.15.1
2. ZeroBrane Studio v1.80 (Oct 07 2018)

## 錯誤訊息

1. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/70245142-8fa72400-17b0-11ea-87d1-1ea9dc21522b.png)

2. 訊息內容

    ```txt
    無法打開應用程式『ZeroBraneStudio』
    ```

## 確認問題發生原因

1. 使用 bash 開啟 ZeroBraneStudio

    ```bash
    open /Applications/ZeroBraneStudio.app
    ```

2. 出現 `-10810` error

    ```txt
    LSOpenURLsWithRole() failed with error -10810 for the file /Applications/ZeroBraneStudio.app.
    ```

    ![2newerror](https://user-images.githubusercontent.com/3851540/70245143-8fa72400-17b0-11ea-96c8-1aca1a1e265d.png)

3. 問題發生原因

    從 macOS Catalina 開始，32 位元 app 將不再與 macOS 相容，而目前 ZeroBraneStudio 官網提供下載的版本仍為 32bit

## 如何解決

從官方 GitHub 的 issue - [MacOS Catalina - fails to start](https://github.com/pkulchenko/ZeroBraneStudio/issues/1020) 翻到解決方式如下

1. clone `all-binaries-190b` branch

    ```bash
    git clone https://github.com/pkulchenko/ZeroBraneStudio.git -b all-binaries-190b
    ```

2. 直接執行 `zbstudio.sh`

    ```bash
    cd ZeroBraneStudio && sh zbstudio.sh
    ```

3. 成功開啟

    ![3success](https://user-images.githubusercontent.com/3851540/70245144-8fa72400-17b0-11ea-9462-19e1746c5113.png)

## 心得

我從 issue [MacOS Catalina - fails to start](https://github.com/pkulchenko/ZeroBraneStudio/issues/1020) 看到官方在 10/25 說會不久即將更新，但過了一個多月還是沒看到，不知道是卡在哪邊，不過既然已經有個可以 work 的 branch，我想應該只是流程上的問題吧？！

## 參考資訊

1. [MacOS Catalina - fails to start](https://github.com/pkulchenko/ZeroBraneStudio/issues/1020)
