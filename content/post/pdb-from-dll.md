---
title: "從 DLL 中建立 PDB 檔"
date: 2018-05-20T19:14:00+08:00
lastmod: 2018-10-06T19:14:40+08:00
draft: false
tags: ["Debug","Tools"]
slug: "pdb-from-dll"
aliases:
    - /2018/05/pdb-from-dll.html
---
# 從 DLL 中建立 PDB 檔
這是在追查某個 dll 可能的潛在效能問題時，延伸出的問題，一般情況下我們透過 NuGet 下載套件時大多都沒有包含 PDB - Program DataBase file 檔，讓追查錯誤時少了一個好用有效的利器，所以才興起從 dll 取得 pdb 的念頭，google 後在 `stackoverflow` 上看到可以透過 `dotPeek` 來製作 PDB，立馬動手實驗看看並紀錄一下

## 關於 dotPeek 
- 是知名軟體開發商 JetBrains，所推出的免費 .NET Decompiler and Assembly Browser
- 支援反組譯 `.dll`, `.exe`, `.winmd` 與 `.baml`
- 支援開啟 `.zip`, `.vsix` 與 `.nupkg`
- 可經由指定 `.pdb` 檔案或是 source 伺服器下載 pdb 以方便偵錯
- 可以產生 PDB 檔
- 可以將反組譯的結果儲存為 Visual Studio 的專案
- [Download dotPeek](https://www.jetbrains.com/decompiler/download/) 可以下載安裝檔

## 從 DLL 建立 PDB 檔
1. 下載並安裝 `dotPeek`
    
    > [Download dotPeek](https://www.jetbrains.com/decompiler/download/) 可以下載安裝檔 
2. 開啟目標 DLL
    
    > 這邊使用 EntityFramework 6.2.0 使為範例 
    - File --> Open...
        
        ![1open](https://user-images.githubusercontent.com/3851540/40278305-334ebeae-5c61-11e8-9bb3-9afc4264cace.png)
3. 產生 PDB
    - 在目標 assembly 上按右鍵 --> Generate Pdb...
        
        ![2generatepdb](https://user-images.githubusercontent.com/3851540/40278306-337b3060-5c61-11e8-8b06-d90c4b3def8d.png)
    - 選擇 PDB 存放資料夾
        
        ![3desfolder](https://user-images.githubusercontent.com/3851540/40278307-33a5fa5c-5c61-11e8-9fe2-0aea6629e4d0.png)
4. 產出結果
    - 會產生 `EntityFramework.pdb`/`990FA7A336B74936B515FEF0C4D65C751` 資料夾
    - `.pdb` 置於 `{dll path}/{dll name}.pdb/{Assembly hash}/` 中
        
        ![4output](https://user-images.githubusercontent.com/3851540/40278308-33cf37dc-5c61-11e8-9624-90459a1da7bd.png)

## 心得
JetBrains 真是愈來愈令人敬佩呀，從 Visual Studio 的強大擴充套件 - `ReSharper`、前端便利 IDE - `WebStorm`、python 少數支援 behave 的 IDE - `PyCharm`、 跨平台的 .net 開發 IDE - `Rider` 到 Google 欽點的新 Android 開發語言 - `Kotlin`，每項工具都非常好用，讓我每次只要有想嘗試的工具都會先上 JetBrains 挖挖寶

這次的需求：從 DLL 中建立 PDB 檔，透過 `dotPeek` 簡直稱得上是無腦操作呀，流程上簡單又直覺，再次讚嘆 JetBrains 的強大，另外值得一提的是 `dotPeek` 是免費工具，非常佛心值得推薦呀

# 參考資訊
1. [DOTPEEK FEATURES](https://www.jetbrains.com/decompiler/features/)
2. [How can I get a PDB file for the EntityFramework NuGet package?](https://stackoverflow.com/questions/32104912/how-can-i-get-a-pdb-file-for-the-entityframework-nuget-package)