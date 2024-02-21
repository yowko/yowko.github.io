---
title: "從 macOS 中移除 .NET Runtime 與 SDK"
date: 2024-02-21T00:30:00+08:00
lastmod: 2024-02-21T00:30:31+08:00
draft: false
tags: ["dotnet","macOS"]
slug: "remove-dotnet-sdk-runtime-from-mac"
---

## 從 macOS 中移除 .NET Runtime 與 SDK

之前筆記 [從 Mac 移除 .NET Core Runtime 與 SDK](/remove-dotnet-from-mac/) 紀錄到因為團隊定期升級 .NET 版本，所以順手紀錄如何移除舊版本的 .NET Core SDK，隨著時間的推移，中間又升級了好幾次，但這次升級 .NET 8 時卻覺得過去這候筆記記有缺陷(會刪掉所有 .NET Runtime 與 SDK，但我其實同時需要兩個大版)，因此就來重新整理一下

## 基本環境說明

1. macOS Sonoma 14.3.1
2. .NET 6.0.418

## 刪除方式

1. 刪除 .NET SDK

    > 以 `6.0.418` 為例

    {{<gist yowko 7dcad66803d517590b1d096f127f39f4 "remove-sdk.sh">}}

2. 刪除 .NET Runtime

    > 以 `6.0.26` 為例

    {{<gist yowko 7dcad66803d517590b1d096f127f39f4 "remove-runtime.sh">}}

## 心得

- 刪除前

    ![1before](https://github.com/yowko/picsbed/assets/3851540/27a74f43-4deb-4a88-a6bd-a855f4a63c03)

- 刪除後

    ![2after](https://github.com/yowko/picsbed/assets/3851540/6363824c-9834-4c2a-bfa3-b569aef16cfc)

直接刪除的做法比較直覺，也比較不容易出錯，跟過去的刪除所有 dotnet 相關套件比較起來，才能保留其他版本的 .NET SDK，不過這樣的做法也有一個缺點，就是如果要刪除多個版本的 .NET SDK，就要執行多次指令

## 參考資訊

1. [從 Mac 移除 .NET Core Runtime 與 SDK](/remove-dotnet-from-mac/)
2. [Removing .NET SDKs from MacOS Manually](https://gist.github.com/justinyoo/68c11fa00480fa15d38f7288815c6ba5)
3. [Microsoft Learn:How to remove the .NET Runtime and SDK](https://learn.microsoft.com/en-us/dotnet/core/install/remove-runtime-sdk-versions?WT.mc_id=DOP-MVP-5002594)
