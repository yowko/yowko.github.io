---
title: "在 Debian container 中安裝 .NET Core SDK"
date: 2020-04-13T22:30:00+08:00
lastmod: 2020-09-01T22:30:31+08:00
draft: false
tags: ["Container","Linux","dotnet core"]
slug: "debian-container-install-dotnet-core-sdk"
---

## 在 Debian container 中安裝 .NET Core SDK

之前筆記 [在 Debian container 中無法成功註冊微軟金鑰](https://blog.yowko.com/debian-container-broken-pipe) 提到想要在 container 中利用 dotnet cli 做些測試，但經過 `multi-stage builds` 來建立 image 中只有 .NET Core runtime 沒有需要的 dotnet cli，所以今天就來紀錄一下在 Debian container 中安裝 .NET Core SDK 的步驟

## 基本環境說明

1. mcr.microsoft.com/dotnet/core/runtime:3.1.3
2. .NET Core SDK 3.1.201

## 安裝步驟

1. 更新 apt-get 並安裝 `wget` 與 `gpg`

    ```bash
    apt-get update
    apt-get install -y wget gpg
    ```

    >如果沒安裝 `gpg` 會造成後續註冊 key 的動作失敗，詳細內容可以參考前筆記 [在 Debian container 中無法成功註冊微軟金鑰](https://blog.yowko.com/debian-container-broken-pipe)

2. 註冊微軟金鑰

    ```bash
    wget -O- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
    mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
    ```

3. 註冊產品存儲庫

    ```bash
    wget https://packages.microsoft.com/config/debian/10/prod.list
    mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
    ```

4. 更新 apt-get 並安裝 .NET Core SDK

    ```bash
    apt-get update
    apt-get install -y apt-transport-https dotnet-sdk-3.1
    ```

## 心得

之前安裝時卡在 `gpg` (詳細內容可以參考前筆記 [在 Debian container 中無法成功註冊微軟金鑰](https://blog.yowko.com/debian-container-broken-pipe))，一排除後大致上就沒問題了，我將 [Debian 10 Package Manager - Install .NET Core](https://docs.microsoft.com/zh-tw/dotnet/core/install/linux-package-manager-debian10?WT.mc_id=DOP-MVP-5002594) 上的步驟做了些省略(能省則省，但除了 `1` 之外，其他動作都可做可不做)：

1. container 中不能使用 sudo
2. 權限調整可以跳過
3. 將多行指令合併成單行

## 參考資訊

1. [在 Debian container 中無法成功註冊微軟金鑰](https://blog.yowko.com/debian-container-broken-pipe)
2. [Debian 10 Package Manager - Install .NET Core](https://docs.microsoft.com/zh-tw/dotnet/core/install/linux-package-manager-debian10?WT.mc_id=DOP-MVP-5002594)
