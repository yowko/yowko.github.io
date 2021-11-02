---
title: "快速更換 mac 中的所有 git repository 位置"
date: 2020-12-22T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["macOS","git"]
slug: "mac-change-git-url"
---

## 快速更換 mac 中的所有 git repository 位置

最近團隊移動了 git server 的位置，所以 git repository 的 url 也就跟著改變了，只是這麼一來團隊那些數十個 repository 要逐一修改顯得有些浪費時間，所以心一橫直接用 bash 換掉，雖然只是句語法，但類似需求已經不是第一次遇到，依國際慣例就是要來紀錄一下囉

## 基本環境說明

1. macOS Catalina 10.15.7
2. git version 2.27.0

## 設定方式

1. 語法

    ```bash
    for d in */**/.git/config ; do
    sed -i "" 's/{舊 url}/{新的 url}/g' $d
    done
    ```

2. 範例

    ```bash
    for d in */**/.git/config ; do
    sed -i "" 's/git@gitlab.old.com/git@gitlab.new.com/g' $d
    done
    ```

- 額外補充
  
    > `sed -i` 在 mac 上有基本的保護機制：修改檔案前需要先做備份，但備份檔可以給空字串，這就是上述語法 `sed -i` 接著是 `""` 的原因

  - 錯誤訊息

    ```bash
    sed: 1: ".git/config": invalid command code .
    ```

  - 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/102957393-4473c680-4515-11eb-9689-551787ba0573.png)

## 心得

概念上很單純，需要的就是知道 git repository url 的設定位置，另外就是在 mac 上 `sed` 語法的特性了解

## 參考資訊

1. [mac 下sed命令的-i参数](https://blog.csdn.net/dawn_moon/article/details/8547408)
