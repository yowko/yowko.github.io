---
title: "使用 Selenium IDE 與 C# 做 Web UI 測試"
date: 2017-06-05T21:00:00+08:00
lastmod: 2021-10-26T16:33:39+08:00
draft: false
tags: ["C#","NUnit","TDD","Unit Test"]
slug: "selenium-ide-csharp-web-ui-test"
aliases:
    - /2017/06/selenium-ide-c-sharp-web-ui-test.html
    - /2017/06/selenium-ide-csharp-web-ui-test/
---
## 使用 Selenium IDE 與 C# 做 Web UI 測試

網頁 UI 及前端技術變化很快，也愈來愈專業，分工愈來愈細，除了前後端獨立的 unit test 之外，前後端介接完成後的整合測試也是重點之一也因為更貼近使用者，測試的真實性及重要性更高。使用者看見錯誤畫面時不會在意前端或是後端單元測試是否正常，使用者只覺得我們沒做好，所以不論錯誤的嚴重性，只要出現錯誤畫面都會造成商譽損失、使用者信任度降低。

除了前後端的整合驗證外， Web UI 測試也很適合用來針對還沒有落實 unit test 的團隊做基本保護：針對較重要或是使用率較高的操作流程進行 Web UI 的自動化測試，在每次更版或是每天，甚至是覺得網站異常時執行自動化測試，確認結果是否合乎預期，這樣一來網站至少開始有了基本的保障，接著只要畫面或是程式被改壞時，馬上就知道，避免明顯錯誤直接在使用者面前裸奔

雖然知道 Selenium IDE 好幾年了，但 Firefox 47 版開始跟 Selenium IDE 並不相容，搞了好久，上 TDD 課程聽 91 大提到 Firefox 53(2017/4/19) 之後問題已獲得解決，備忘一下順便祈禱不會再出現相同的問題了

## 下載 Selenium IDE

> Selenium IDE 目前只支援 Firefox

* 請使用 `Firefox`

* 開啟`附加元件`管理員

    ![1plugin](https://cloud.githubusercontent.com/assets/3851540/26777306/4df08b3a-4a0f-11e7-93f1-904b29b0a811.png)

* 取得元件 --> 看看更多附加元件

    ![2moreplugin](https://cloud.githubusercontent.com/assets/3851540/26777307/4df13396-4a0f-11e7-87db-1a64ebf7b99b.png)

* 搜尋 `selenium ide` --> 切換至 `Most Users` --> 安裝 `Selenium IDE`

    > 搜尋 `selenium ide` 會出現很多 Add-ons ，切換至 `Most Users` 比較容易找到真正需要的，附上 url：[Add-ons - Selenium IDE](https://addons.mozilla.org/en-US/firefox/addon/selenium-ide/)

    ![3search](https://cloud.githubusercontent.com/assets/3851540/26777308/4df33b28-4a0f-11e7-9a68-3cf908b4aae2.png)

* 重新啟動 Firefox

    ![4restart](https://cloud.githubusercontent.com/assets/3851540/26777309/4df36800-4a0f-11e7-8a05-960022eca402.png)

## 啟動 Selenium IDE

1. 預設隱藏選單

    ![5hide](https://cloud.githubusercontent.com/assets/3851540/26777310/4df5bf56-4a0f-11e7-8339-285408871f4f.png)

2. 開啟選單 --> 工具 --> Selenium IDE 或是直接按下快捷鍵(Ctrl + Alt + S)

    > 按下 `alt` 可以開啟選單

    ![6startseleniumide](https://cloud.githubusercontent.com/assets/3851540/26777311/4e078f2e-4a0f-11e7-8c48-13fe2359939c.png)

## 開始錄製網頁操作

1. Selenium IDE 開啟時預設已啟動錄製

    ![7record](https://cloud.githubusercontent.com/assets/3851540/26777312/4e14f01a-4a0f-11e7-8411-ab07e484e4ff.png)

2. 直接開啟目標網站並執行需要的操作
    * 開啟 GitHub

        ![8opengithub](https://cloud.githubusercontent.com/assets/3851540/26777313/4e198ecc-4a0f-11e7-857b-746bbed5dccf.png)

    * 按下 Sign in

        ![9opensignin](https://cloud.githubusercontent.com/assets/3851540/26777315/4e1b5536-4a0f-11e7-9651-f4222fdc3dfc.png)

    * 輸入帳號密碼
    * 執行登入

        ![10fillup](https://cloud.githubusercontent.com/assets/3851540/26777316/4e1c1778-4a0f-11e7-90b2-1c3e7fb1621f.png)

3. 錄製腳本後可以隨時執行測試

    ![8autoplay](https://cloud.githubusercontent.com/assets/3851540/26777314/4e19d62a-4a0f-11e7-82e4-3082fcd525b3.gif)

## 如何讓 C# 也能執行自動測試

> 如果需要有 Selenium IDE 才能執行自動化測試，不免增加使用困難度，這時候我們就可以將 Selenium IDE 錄製的結果轉換為 C# 程式

1. Selenium IDE 選單 --> 檔案 --> Export Test Suite As --> C# / NUnit /WebDriver

    ![11export](https://cloud.githubusercontent.com/assets/3851540/26777317/4e30f85a-4a0f-11e7-8773-22a10438eebc.png)

2. 記得存成 `.cs` 檔

    ![12save](https://cloud.githubusercontent.com/assets/3851540/26777300/4dc86fce-4a0f-11e7-8df9-13abc4e37ff5.png)

3. 使用 Visual Studio 開啟匯出的檔案
4. 加入 NuGet 套件

    * Selenium.WebDriver

        ![13seleniumwebdriver](https://cloud.githubusercontent.com/assets/3851540/26777302/4dcee3e0-4a0f-11e7-99aa-2ff4816eb053.png)

    * Selenium.Support

        ![14seleniumsupport](https://cloud.githubusercontent.com/assets/3851540/26777303/4dcfb14e-4a0f-11e7-81d5-763128dcf0db.png)

    * 需要瀏覽器 driver
        * firefox

            ![16ff](https://cloud.githubusercontent.com/assets/3851540/26777304/4dd0af72-4a0f-11e7-8760-6b97eaff686c.png)

        * chrome

            ![17chrome](https://cloud.githubusercontent.com/assets/3851540/26777305/4ddc813a-4a0f-11e7-8a13-8f4add2ceeae.png)

        * 如果沒有安裝瀏覽器 driver 的錯誤訊息

            ![15drivererror](https://cloud.githubusercontent.com/assets/3851540/26777301/4dcdca64-4a0f-11e7-9526-e33c85fd6d26.png)

5. 接著依一般測試方法執行，Visual Studio 就會自行啟動指定瀏覽器進行測試了

## 心得

使用上很簡單的軟體，卻能達成很不錯的保護效果，執行速度也很快，程式有異動時都先執行幾個必要的 Web UI 測試，一旦出現錯誤，就暫緩上線作業，可以有效地讓程式碼品質獲得保障

## 參考資訊

1. [Add-ons - Selenium IDE](https://addons.mozilla.org/en-US/firefox/addon/selenium-ide/)
