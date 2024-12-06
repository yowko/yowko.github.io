---
title: "使用 JetBrains Rider 來 Debug Source Generators 或 Incremental Generators"
date: 2024-12-06T00:30:00+08:00
lastmod: 2024-12-06T00:30:31+08:00
draft: false
tags: ["dotnet","csharp","debug"]
slug: "debug-source-generators-incremental-generators"
---

## 使用 JetBrains Rider 來 Debug Source Generators 或 Incremental Generators

之前筆記 [使用 Source Generators 來為 method 加上時間測量](/source-generators-stopwatch) 與 [使用 Incremental Generators 來為 method 加上時間測量](/incremental-generators-stopwatch) 紀錄到如何使用 Source Generators 與 Incremental Generators 來為 method 加上 stopwatch 測量 method 的執行時間，但 Source Generators 與 Incremental Generators 的專案特性、語法都與一般專案不同，當然 debug 方式也不一樣，今天就來記錄如何使用 JetBrains Rider 來 Debug Source Generators 或 Incremental Generators。

今天主要是紀錄如何 debug Source Generators 或 Incremental Generators 產生檔案的過程。

對產生出的檔案 debug 的方發，原則上跟一般專案的 debug 相同就不特別紀錄，前提是

1. 先確定是否有正確執行，並且產生出預期的檔案
2. 只有 `netstandard2.0` 或是 `netstandard2.1` 會在 Dependencies 中出現

## 基本環境說明

- macOS Sequoia 15.1.1 (Apple M2 Pro)
- dotnet sdk 9.0.100
- JetBrains Rider 2024.3
- NuGet package

    - Microsoft.CodeAnalysis.CSharp 4.12.0

- 使用專案：[GitHub:yowko/source-generators-stopwatch](https://github.com/yowko/source-generators-stopwatch)

## 操作方式

1. 新增 `Properties/launchSettings.json`

    雖然 [JetBrains 官方文件:Debug Source Generators in JetBrains Rider](https://blog.jetbrains.com/dotnet/2023/07/13/debug-source-generators-in-jetbrains-rider/) 需要將 `Properties/launchSettings.json` 放在 Source Generators 專案中，但我測試發現放在實際引用的專案也可正常啟動 debug

    ![0officialdoc](https://github.com/user-attachments/assets/50bc0368-280e-4fbf-84f6-fbb529130a31)

    {{<gist yowko 4a9c8baf8d2a4bdca1c949ccb1462cfa "launchSettings.json">}}

2. 執行 debug

    - 可以從 launchSettings.json 側邊欄啟動

        ![1launchsetting](https://github.com/user-attachments/assets/7c827eb2-b3e5-4b9c-87d0-2f3ca8e461da)

    - 從 `Run Widget` 啟動

        > 我第一次沒有出現在 `Run Widget`，我從 launchSettings.json 側邊欄啟動後才出現

        ![2runwidget](https://github.com/user-attachments/assets/6a83d6f4-e850-4225-a3c6-b10135ffded3)

## 心得

我自己測試的結論是

1. `Properties/launchSettings.json` 放在 Source Generators 專案或是實際引用的專案都可以正常啟動 debug
2. `Properties/launchSettings.json` 只是啟動 debugger，但實際 debug 的專案還是看執行專案所引用的是哪一個

    在 `SourceGenerators_ns21` 專案的 `Properties/launchSettings.json` 啟動 debug，但 `SourceGeneratorsDemo` 專案設定的 ProjectReference 是 `SourceGenerators_ns20`，所以實際 debugger 是針對 `SourceGeneratorsDemo` 專案進行 debug

## 參考資料

1. [使用 Source Generators 來為 method 加上時間測量](/source-generators-stopwatch)
2. [使用 Incremental Generators 來為 method 加上時間測量](/incremental-generators-stopwatch)
3. [Debug Source Generators in JetBrains Rider](https://blog.jetbrains.com/dotnet/2023/07/13/debug-source-generators-in-jetbrains-rider/)