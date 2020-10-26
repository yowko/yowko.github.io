---
title: "避免執行 dnf update 時連帶升級特定套件"
date: 2020-10-26T21:30:00+08:00
lastmod: 2020-10-26T21:30:31+08:00
draft: false
tags: ["Linux"]
slug: "dnf-update-exclude"
---

## 避免執行 dnf update 時連帶升級特定套件

之前對於 linux 不熟悉，加上沒有系統化學習而對 linux 相關知識不足，造成不少問題：常常要安裝新套件前，就執行 `yum update`，以為這個指令是讓 yum 重新取得 package cache，後來才發現這個指令會升級系統上所有套件至最新版 XD

雖然學藝不精下錯指令是我個人問題，但為了避免有人像我一樣，還是應該強制避免某些套件在不預期的情況被升級，快速紀錄一下  提醒自己別再犯基本錯誤 QQ

## 基本環境說明

1. Azure VM Standard_B2s (CentOS-based 8.2 - Gen1)
2. Linux (centos 8.2.2004)
3. Linux Kernel 4.18.0-193.19.1.el8_2.x86_64
4. dnf 4.2.17
5. Redis 6.0.7

## 設定方式

1. 設定前狀況：執行 dnf update 會將 redis 從 Redis 6.0.7 升級至 Redis 6.0.8

    > `dnf list installed` 可以列出已安裝的套件

    ![1dnfupdate](https://user-images.githubusercontent.com/3851540/97158431-60c4e280-17b4-11eb-95c5-ba93bfed1e0a.png)

2. 將需要鎖定的套件名稱加至 `/etc/dnf/dnf.conf` 的 `exclude` 中

    > 原本就有設定 exclude，就將 `redis` 補在最後面(如果原本就有 redis，會重複)，如果原本沒有就直接加上 `exclude=redis`

    - 語法

        ```bash
        grep -q '^exclude=' /etc/dnf/dnf.conf && sed -i 's/\(exclude=.*\)/\1 <package> [<package> ...]/g' /etc/dnf/dnf.conf || echo 'exclude=<package> [<package> ...]' >> /etc/dnf/dnf.conf
        ```

    - 範例

        ```bash
        grep -q '^exclude=' /etc/dnf/dnf.conf && sed -i 's/\(exclude=.*\)/\1 redis/g' /etc/dnf/dnf.conf || echo 'exclude=redis' >> /etc/dnf/dnf.conf
        ```

3. 再次更新即可忽略

    ![2afterupfate](https://user-images.githubusercontent.com/3851540/97158436-64586980-17b4-11eb-9959-0689edf7531a.png)

- 備註
  
    dnf exclude 排除特定套件後，dnf 的所有操作都會直接忽略，像是 dnf list installed 也不會列出該套件

    ![3exclude](https://user-images.githubusercontent.com/3851540/97158438-64f10000-17b4-11eb-8546-f3988d239ff5.png)

## 心得

感覺上在設定 dnf exclude 時，語法不夠漂亮：可能重複設定，但目前功力需要多點時間再試試，就待有空或是下次遇到再一起看囉，重點還是設定 `/etc/dnf/dnf.conf` 的 `exclude` 來達成排除特定套件的檢查

另外是設定 `exclude` 後， `dnf list installed` 就無法列出設定的套件資訊，有時候要查是否安裝或是版本資訊時  需要特別留意

## 參考資訊

1. [Fedora 24: Exclude package from update](https://www.hiroom2.com/2016/07/07/fedora-24-exclude-package-from-update/)
2. [規則運算式中的反向參考建構](https://docs.microsoft.com/zh-tw/dotnet/standard/base-types/backreference-constructs-in-regular-expressions?WT.mc_id=DOP-MVP-5002594)
3. [sed backreference: get each line and append it to end of line](https://unix.stackexchange.com/a/265515)
4. [How to add a line in sed if not match is found](https://stackoverflow.com/a/20268787)
5. [10 Yum Exclude Examples to Skip Packages for Linux Yum Update (How to Yum Exclude Kernel Updates)](https://www.thegeekstuff.com/2014/11/yum-exclude-examples/)
