---
title: "在 Debian container 中無法成功註冊微軟金鑰"
date: 2020-04-13T21:30:00+08:00
lastmod: 2020-09-01T21:30:31+08:00
draft: false
tags: ["Container","Linux"]
slug: "debian-container-broken-pipe"
---

## 在 Debian container 中無法成功註冊微軟金鑰

最近部署在 debian base 中的 .NET Core application 有些異常，所以打算在 container 中使用 dotnet cli 做些測試，不過身為追求 Container 優化的工程師，一定是使用 `multi-stage builds` 來建立 image，所以 container 中僅有 .NET Core runtime 而沒有包含 dotnet cli 的 SDK，今天要紀錄的就是在安裝 .NET Core SDK 時遇到的問題

## 基本環境說明

1. mcr.microsoft.com/dotnet/core/runtime:3.1.3
2. .NET Core SDK 3.1.201

## 問題與解決方式

1. 問題描述

    > 執行 `wget -O- https://packages.microsoft.com/keys/microsoft.asc | gpg       --dearmor > microsoft.asc.gpg` 後出現錯誤

    - 錯誤訊息

            ```txt
            root@9f771a7513d9:/# wget -O- https://packages.microsoft.com/keys/microsoft.asc | gpg       --dearmor > microsoft.asc.gpg
            bash: gpg: command not found
            --2020-04-13 02:45:21--  https://packages.microsoft.com/keys/microsoft.asc
            Resolving packages.microsoft.com (packages.microsoft.com)... 23.99.120.248
            Connecting to packages.microsoft.com (packages.microsoft.com)|23.99.120.248|:443...         connected.
            HTTP request sent, awaiting response... 200 OK
            Length: 983 [application/octet-stream]
            Saving to: 'STDOUT'

            -                                                                     0%        [                                                                                                                                                                           ]       0       --.-KB/s    in 0s


            Cannot write to '-' (Broken pipe).
            ```

    - 錯誤截圖

        ![1error](https://user-images.githubusercontent.com/3851540/79090823-85f48500-7d7d-11ea-856e-a2aea9867509.png)

2. 解決方式

    ```bash
    aptt-get install -y gpg
    ```

    > 安裝後即可成功註冊 key

    ![2ok](https://user-images.githubusercontent.com/3851540/79090825-8856df00-7d7d-11ea-8274-14cc12b300dc.png)

## 心得

仔細看錯誤訊息可以發現有一段 `bash: gpg: command not found`，揭露因為沒有安裝 `gpg` 所以 pipe 之後動作也就無法成功執行，連帶造成無法順利安裝 .NET Core SDK

但遇到問題當下，只看到 `Cannot write to '-' (Broken pipe).` google 到的狀況提到大多是權限問題造成的，方向完全不對，當然最後也沒有解決問題，過了幾天重新 review 時才發現根本原因，紀錄一下提醒自己就算再急還是要持冷靜，別慌了手腳

## 參考資訊

1. [.NET Core Runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/)
2. [Debian 10 Package Manager - Install .NET Core](https://docs.microsoft.com/zh-tw/dotnet/core/install/linux-package-manager-debian10?WT.mc_id=DOP-MVP-5002594)
