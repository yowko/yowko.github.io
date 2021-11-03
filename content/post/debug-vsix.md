---
title: "如何 Debug 你做的 Visual Studio 擴充套件(vsix)"
date: 2017-05-31T00:30:00+08:00
lastmod: 2021-11-03T00:30:32+08:00
draft: false
tags: ["Debug","NUnit","Unit Test","Visual Studio"]
slug: "debug-vsix"
aliases:
    - /2017/05/debug-vsix.html
---
## 如何 Debug 你做的 Visual Studio 擴充套件(vsix)

從 bot framework project template 到最近做的 xUnit.net 2.0、NUnit2、NUnit3 test framework，前前後後也上架了四個擴充套件(vsix)，我都有把相關實作紀錄下來：[如何發行 Visual Studio 專案範本(project Template)](/publish-visual-studio-project-template)、[Test Framework 套件不好用嗎？！ 自己做一個囉](/customize-test-framework-project-template)，但前面文章都只紀錄該如何可以成功實作，沒紀錄背後花最多時間的嘗試及摸索，今天就來分享該如何 debug Visual Studio 擴充套件(vsix)

以下會以 [Test Framework 套件不好用嗎？！ 自己做一個囉](/customize-test-framework-project-template) 製作的專案範本做為範例，程式碼在 [NUnit3.TestGenerator](https://github.com/yowko/NUnit3.TestGenerator)

## 開啟具有 vsix 專案的 solution

首先第一步當然就是開啟有 VSIX 專案的方案(solution)，管道有兩個：

1. 自行建立 vsix project

    * 1-1. 方案上按 右鍵 --> Add --> New Project...

        ![4newproject](https://cloud.githubusercontent.com/assets/3851540/26543098/1c554046-448f-11e7-9a09-52576820f960.png)

    * 1-2. Extensibility --> VSIX Project --> 填入方案名稱

        ![12addvsix](https://cloud.githubusercontent.com/assets/3851540/26543106/1c81f352-448f-11e7-8ab1-444cbbed8243.png)

        * 看不到 `Extensibility` 時請重新執行 Visual Studio 2017 安裝程式，並加入 Visual Studio extension development 開發套件

            ![13noextension](https://cloud.githubusercontent.com/assets/3851540/26543107/1c9a3200-448f-11e7-8b65-8c563d707d6b.png)

            ![14noextension2](https://cloud.githubusercontent.com/assets/3851540/26543089/1c24dd48-448f-11e7-8e5f-7afb04de7203.png)

            ![15noextension3](https://cloud.githubusercontent.com/assets/3851540/26543092/1c29382a-448f-11e7-8f3e-a59f0e3088a9.png)

2. 使用已有 vsix project 的 solution

## 設定 vsix 專案

1. vsix 專案 --> 按右鍵 --> Properties

    ![1setupproject](https://cloud.githubusercontent.com/assets/3851540/26576431/7cbdf8c0-455b-11e7-892c-c9a6089ffacd.png)

2. Debug

    * Start external program

        > 設定啟動的 debug 程式 (我使用 vs2017：`C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\devenv.exe`)

        ![2vs2017](https://cloud.githubusercontent.com/assets/3851540/26576160/6d84337a-455a-11e7-9f45-30f247945d43.png)

    * Start options

        > 設定啟動參數：`/rootsuffix Exp`

        ![3startoption](https://cloud.githubusercontent.com/assets/3851540/26576161/6d84e46e-455a-11e7-9e85-237bf7635018.png)

## 將 vsix 設為啟動專案

* vsix 專案 --> 按右鍵 --> Set as StartUp Project

    ![4startup](https://cloud.githubusercontent.com/assets/3851540/26576163/6d8ab650-455a-11e7-8b89-710515958b34.png)

## 直接按下 F5 進行 debug

會以 experimental instance 方式開啟另一個 Visual Studio 並已安裝啟動的 vsix，不會影響平常使用的 Visual Studio，但針對 experimental instance 安裝的套件會一直留在 experimental instance 中，不會自行移除，這點需要特別留意

> experimental instance 的相關資訊可以參考 [The Experimental Instance](https://msdn.microsoft.com/en-us/library/bb166560.aspx)

* experimental instance Visual Studio 在標題列除了一般的專案名稱外還會有 `Experimental Instance` (實驗執行個體)的字樣

    ![5ExperimentalInstance](https://cloud.githubusercontent.com/assets/3851540/26576159/6d847588-455a-11e7-945a-71a6ff9ad9dd.png)

## 實際效果

![6debugging](https://cloud.githubusercontent.com/assets/3851540/26576953/3ffaf026-455d-11e7-9cc3-9810a22ad1e9.gif)

## 參考資訊

1. [如何發行 Visual Studio 專案範本(project Template)](/publish-visual-studio-project-template)
2. [Test Framework 套件不好用嗎？！ 自己做一個囉](/customize-test-framework-project-template)
3. [NUnit3.TestGenerator](https://github.com/yowko/NUnit3.TestGenerator)
4. [How to debug a Vsix project](https://stackoverflow.com/questions/24653486/how-to-debug-a-vsix-project)
5. [The Experimental Instance](https://msdn.microsoft.com/en-us/library/bb166560.aspx)
