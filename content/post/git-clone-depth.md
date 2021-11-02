---
title: "加快大型 GIT Repository 下載速度(指定 depth)"
date: 2018-04-07T16:33:00+08:00
lastmod: 2021-11-02T16:33:16+08:00
draft: false
tags: ["Git"]
slug: "git-clone-depth"
aliases:
    - /2018/04/git-clone-depth.html
---
## 加快大型 GIT Repository 下載速度(指定 depth)

同事反應有個專案在經年累月的發展下，快速成長到每次 clone 都要個十幾二十分鐘，後來甚至有次在 production deploy 時還完全卡死，無法正確 pull 新的 code

在檢查無法 pull 的過程中，發現光只是 clone 就讓我失去耐心了，所以分享給同事只抓最後版本的做法(發現用自己筆記分享做法給同事超省事還能 reuse ，超棒的 哈哈)

## 語法及說明

- git clone 語法

    ```cmd
    git clone [--template=<template_directory>]
      [-l] [-s] [--no-hardlinks] [-q] [-n] [--bare] [--mirror]
      [-o <name>] [-b <name>] [-u <upload-pack>] [--reference <repository>]
      [--dissociate] [--separate-git-dir <git dir>]
      [--depth <depth>] [--[no-]single-branch] [--no-tags]
      [--recurse-submodules[=<pathspec>]] [--[no-]shallow-submodules]
      [--jobs <n>] [--] <repository> [<directory>]
    ```

- `--depth <depth>`

    ```log
    Create a shallow clone with a history truncated to the specified number of commits. Implies --single-branch unless --no-single-branch is given to fetch the histories near the tips of all branches. If you want to clone submodules shallowly, also pass --shallow-submodules.
    ```

    > 僅會下載指定數量的歷史紀錄，隱含 `--single-branch` 指令效果，除非明確加上 `--no-single-branch`，針對 `submodule` 則可使用 `--shallow-submodules`

## git clone 時加上 `--depth` 參數

以下使用 linux kernel 當做範例(因為它夠大)：<https://github.com/torvalds/linux.git>

>下列方法，擇一即可

- 使用指令

    ```cmd
    git.exe clone --depth 1 "https://github.com/torvalds/linux.git" "D:\testGit\linux"
    ```

  - 指定只取得最近 `1` 次 commit 資訊

- 使用 TortoiseGit

    >![1tortoisedepth](https://user-images.githubusercontent.com/3851540/38238485-91d9f3a0-375c-11e8-81b4-c6ee99b2bb35.png)

## 實際效果

- commit log
  - 加上 `--depth 1`

    >![2depth1](https://user-images.githubusercontent.com/3851540/38238486-92089c1e-375c-11e8-8b19-97c2b7da52b2.png)

  - 取得完整歷史紀錄

    >![3fullhistory](https://user-images.githubusercontent.com/3851540/38238488-922f7bd6-375c-11e8-973c-cf4e0990b101.png)

- 大小比較
  - 加上 `--depth 1`：963 MB

    >![4depth](https://user-images.githubusercontent.com/3851540/38238490-925b7998-375c-11e8-8bb3-c0823c8f2430.png)

  - 取得完整歷史紀錄：2.92 GB

    >![5full](https://user-images.githubusercontent.com/3851540/38238492-9284ac00-375c-11e8-8af4-35d62329cd5c.png)

- 下載時間(誤差請參考心得說明)
  - 加上 `--depth 1`：10m36s
  - 取得完整歷史紀錄：1h11m56s

## 心得

針對下載耗用時間部份，我測試過兩次，兩次誤差非常大，主要的原因是`網路速度`，所以時間的耗用比較適合用來表示一個相對趨勢而非絕對的倍數關係

透過 `repository 使用 size` 或是 `log 數量` 搭配 clone 時間 可以明顯比較出有沒有加入 `--depth` 參數的差異，也可以依據不同目的採用不同策略：只需要 `部份紀錄` 或是想要節省處理時間時，就使用 `--depth`

## 參考資訊

1. [git-clone](https://git-scm.com/docs/git-clone)
2. [linux](https://github.com/torvalds/linux.git)
