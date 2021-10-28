---
title: "在 Visual Studio 2017 中安裝其他 Test Framework - xUnit.net 2.0"
date: 2017-05-29T00:34:00+08:00
lastmod: 2021-10-14T11:55:15+08:00
draft: false
tags: ["Unit Test","Visual Studio","xUnit"]
slug: "visual-studio-2017-test-framework-xunit"
aliases:
    - /2017/05/visual-studio-2017-test-framework-xunit.html
---
## 在 Visual Studio 2017 中安裝其他 Test Framework - xUnit.net 2.0

文章 [在 Visual Studio 2017 中安裝其他 Test Framework - NUnit](/visual-studio-2017-test-framework-nunit) 中測試安裝不同的 Test Framework 時，發現在 Visual Studio 2017 中竟然無法直接使用 xUnit 專案範本 create unit test，著實讓我驚訝不已，本著工程師的浪漫，剛好利用端午連假的空閒，於是興起了動手自己做的念頭

原本以為是件容易的事，想不到也讓我試了一整天才完成，過程改天再另文撰寫，就先來看看如何在 Visual Studio 2017 安裝 xUnit.net 2.0 的 test framework 專案範本

## 安裝 xUnit

* Visual Studio 2017 主選單 --> Tools --> Extensions and Updates...

    ![4extension](https://cloud.githubusercontent.com/assets/3851540/26522005/5e87c2de-4329-11e7-99d9-000de2ba680e.png)

* 安裝 `xUnit.net.TestGenerator`

  * 點擊 `Online` --> 搜尋 `xUnit` --> 下載 `xUnit.net.TestGenerator`

        ![1xunitextension](https://cloud.githubusercontent.com/assets/3851540/26530332/b417c8ca-4405-11e7-9b59-30fe81237c6b.png)

* 安裝後需手動重啟 Visual Studio 2017

    ![2needrestart](https://cloud.githubusercontent.com/assets/3851540/26530333/b43a5520-4405-11e7-94b4-acddcf9c6ad1.png)

* 手動關閉 Visual Studio 2017 會開始啟動安裝

    ![3install1](https://cloud.githubusercontent.com/assets/3851540/26530334/b43fbeca-4405-11e7-9880-3e70d012c7ff.png)

    ![4install2](https://cloud.githubusercontent.com/assets/3851540/26530335/b4581484-4405-11e7-8036-021d6eb2d37f.png)

    ![5install3](https://cloud.githubusercontent.com/assets/3851540/26530337/b45bae64-4405-11e7-9b65-170a19c30817.png)

    ![6install4](https://cloud.githubusercontent.com/assets/3851540/26530336/b459f984-4405-11e7-8190-e7b222f1b9e9.png)

## 使用 xUnit 建立 Unit Test

![7preview](https://cloud.githubusercontent.com/assets/3851540/26530331/b41425da-4405-11e7-98a8-4526dcac7767.png)

## 注意事項

1. 暫不支援 `IntelliTest`

    > 這個功能好像沒有很多人用，有收到回饋需要時再補

2. 無法自訂 Namespace、Test Class format、Test Method format 問題

    > 本來想一併解決這個 bug，試了好久發現 MSTest 也有相同問題，可能是更底層的 bug，也只能暫時忽略了

3. xUnit.net.TestGenerator 的 Marketplace 連結：[xUnit.net.TestGenerator](https://marketplace.visualstudio.com/items?itemName=YowkoTsai.xUnitnetTestGenerator)

    > 有問題可以直接發 issue

## 參考資訊

1. [在 Visual Studio 2017 中安裝其他 Test Framework - NUnit](/visual-studio-2017-test-framework-nunit)
2. [xUnit.net.TestGenerator](https://marketplace.visualstudio.com/items?itemName=YowkoTsai.xUnitnetTestGenerator)
