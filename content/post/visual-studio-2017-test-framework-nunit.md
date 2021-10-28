---
title: "在 Visual Studio 2017 中安裝其他 Test Framework - NUnit"
date: 2017-05-28T00:30:00+08:00
lastmod: 2021-10-14T00:30:11+08:00
draft: false
tags: ["NUnit","Unit Test","Visual Studio"]
slug: "visual-studio-2017-test-framework-nunit"
aliases:
    - /2017/05/visual-studio-2017-test-framework-nunit.html
---
## 在 Visual Studio 2017 中安裝其他 Test Framework - NUnit

透過 Visual Studio 2017 建立專案後，可以使用的 Test Framework 預設只有 MSTest 與 MSTestv2，而 MSTest 一般普遍認為功能較陽春，而會想改使用 NUnit 或是 xUnit，順手筆記留個紀錄

2017/05/17 xUnit.net 尚未支援 Visual Studio 2017 直接使用專案範本建立測試

## 預設只有 MSTest 與 MSTestv2

![1default](https://cloud.githubusercontent.com/assets/3851540/26522001/5e5ed2c0-4329-11e7-9d51-0c09ba592d76.png)

## 安裝 NUnit

* Visual Studio 2017 主選單 --> Tools --> Extensions and Updates...

    ![4extension](https://cloud.githubusercontent.com/assets/3851540/26522005/5e87c2de-4329-11e7-99d9-000de2ba680e.png)

* 安裝以下兩個套件(順序不重要)

    ![10extensions](https://cloud.githubusercontent.com/assets/3851540/26522025/c678659c-4329-11e7-917f-67869288d2c9.png)

  * `Test Generator NUnit extension`

        > 讓 Create Unit Tests 時可以選擇 NUnit

    * 會同時安裝 NUnit2 及 3

            ![2testframework](https://cloud.githubusercontent.com/assets/3851540/26522000/5e5d5012-4329-11e7-8fb9-cb14117b231a.png)

  * `NUnit 3 Test Adapter`

        > 執行 NUnit 測試

    * 如果沒裝無法執行測試
      * 錯誤訊息

                > `No tests found to run`

                ![3notests](https://cloud.githubusercontent.com/assets/3851540/26522003/5e67ad46-4329-11e7-9004-689f7e147c2c.png)

* 安裝後需手動重啟 Visual Studio 2017

    ![5needrestart](https://cloud.githubusercontent.com/assets/3851540/26521997/5e1aac12-4329-11e7-9398-325dabd970d2.png)

* 手動關閉 Visual Studio 2017 會開始啟動安裝

    ![6install1](https://cloud.githubusercontent.com/assets/3851540/26521998/5e3f7678-4329-11e7-838c-12cd53630298.png)

    ![7install2](https://cloud.githubusercontent.com/assets/3851540/26522004/5e6f03a2-4329-11e7-9a90-beb147e2ef29.png)

    ![8install3](https://cloud.githubusercontent.com/assets/3851540/26521999/5e5c778c-4329-11e7-8c06-5fdcf5f2f4ef.png)

    ![9install4](https://cloud.githubusercontent.com/assets/3851540/26522002/5e623a46-4329-11e7-8285-0894dee236ef.png)

## xUnit.net

暫時只能在專案中手動加入 xunit 測試程式來解決，尚未推出支援 Visual Studio 2017 直接使用專案範本建立測試專案

手動加入的部份可以參考 [Getting Started with xUnit.net (desktop)](https://xunit.github.io/docs/getting-started-desktop.html)

2017/5/29 更新：可以參考個人拙作：[xUnit.net.TestGenerator](https://marketplace.visualstudio.com/items?itemName=YowkoTsai.xUnitnetTestGenerator)

## 參考資訊

1. [Visual Studio 2015 如何產生 NUnit 或 xUnit 的測試專案](/visual-studio-2015-use-nunit-xunit)
2. [xUnit.net Test Extensions](https://marketplace.visualstudio.com/items?itemName=BradWilson.xUnitnetTestExtensions)
