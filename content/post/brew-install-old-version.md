---
title: "使用 Homebrew 安裝舊版工具"
date: 2019-12-21T20:30:00+08:00
lastmod: 2019-12-21T20:30:31+08:00
draft: false
tags: ["macOS","Tools"]
slug: "brew-install-old-version"
---

## 使用 Homebrew 安裝舊版工具

前幾天筆記 [使用 Homebrew Cask 安裝舊版本軟體](https://blog.yowko.com/homebrew-cask-install-older-version/) 紀錄到使用 Homebrew Cask 來安裝舊版本的 .NET SDK，這幾天在 mac 上測試功能時，為了配合團隊目前技術而需要 Homebrew 安裝舊版本，查了一下資料，感覺是眾說紛紜，決定自己紀錄一下，避免日後又查好久

## 基本環境說明

1. macOS Catalina 10.15.1

## 安裝方式

1. 指定 `.rb`

    > 作法與 [使用 Homebrew Cask 安裝舊版本軟體](https://blog.yowko.com/homebrew-cask-install-older-version/) 相同，詳細容內請直接參考 [使用 Homebrew Cask 安裝舊版本軟體](https://blog.yowko.com/homebrew-cask-install-older-version/)，以下節錄重點

    - 下載需要版本的 Casks 檔案(`*.rb`)

    - 使用下載的 cask 檔案進行安裝

2. 使用版本 formula name (推薦)

    > 以下使用 `mariadb` 為例

    - 使用 `brew info` 看到當前版本為 `10.4.10`

        ![1brewinfo](https://user-images.githubusercontent.com/3851540/71308607-bd8b9a00-2439-11ea-8ee4-bff6513139c0.png)

    - `brew search mariadb@` 列出舊版本

        ![2brewsearch](https://user-images.githubusercontent.com/3851540/71308608-bd8b9a00-2439-11ea-8414-eaad703e4861.png)

    - `brew install mariadb@10.1` 安裝指定舊版本

        ![3installold](https://user-images.githubusercontent.com/3851540/71308609-be243080-2439-11ea-9ab3-f4855ac980b3.png)

## 心得

出乎意料的是透過 `.rb` 的安裝方式在 `Homebrew` 與 `Homebrew Cask` 都通用，這樣一來最起碼可以只記這種方式就能適用兩種安裝情境了

不過 Homebrew 在不同版本間的安裝體驗比 Homebrew Cask 好許多，至少不需要自己在 GitHub 茫茫檔案大海中尋尋覓覓

## 參考資訊

1. [使用 Homebrew Cask 安裝舊版本軟體](https://blog.yowko.com/homebrew-cask-install-older-version/)
2. [Homebrew install specific version of formula? - Simple Workflow](https://stackoverflow.com/a/9832084/3600583)
3. [How to install an older version of a Homebrew package](https://flaviocopes.com/homebrew-install-older-version/)
