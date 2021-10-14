---
title: "無法啟動 Visual Studio"
date: 2017-10-11T22:29:00+08:00
lastmod: 2021-10-14T22:23:18+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-load-menu-problem"
aliases:
    - /2017/10/visual-studio-load-menu-problem.html
---
## 無法啟動 Visual Studio

最近開始使用 ASP.NET Core 開發新專案，過程中不僅要熟悉新的語法跟架構，就連 Visual Studio 也來湊熱鬧，時不時就 crash ，最近幾天更變本加厲完全開不起來XD，嘗試了 [Visual Studio 怪怪的？！ Visual Studio 好慢？！(Visual Studio 的無痕模式)](http://blog.yowko.com/2016/12/visual-studio-safe-mode.html) 與 [右鍵選單 Create Unit Tests (建立單元測試) 選項不見？！](/2017/05/missing-create-unit-tests.html) 都無法解決問題，原本還以為是 Oracle Developer Tool 引起的，最後還是在 Stack Overflow 的幫助下搞定了，就來看問題發生原因吧

## 錯誤訊息

1. 訊息內容

    ```txt
    A problem occurred when loading the microsoft visual studio menu.
    To fix this problem, run 'devenv.exe /resetsettings' from the command prompt.
    Note: this command resets your enviroment settings.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/31422121-4bb61370-ae7e-11e7-95be-e6c18b0366ad.png)

## 問題發生原因及解決方式

* 嘗試下列幾個重新設定的啟動參數都沒有用
    * `/resetskippkgs`
    * `/installvstemplates`
    * `/resetsettings`
    * `/resetuserdata`

* 問題原因：

    > 在 Stack Overflow 上找到問題真正的原因：`環境變數超過 2047 字元`

    * 開啟環境變數編輯工具就有提示錯誤了

        ![2error](https://user-images.githubusercontent.com/3851540/31422122-4bdea8c6-ae7e-11e7-9c00-0a1a33aa5e6e.png)

* 解決方式

    * 依我個人而言，刪除幾個已經沒有用到的設定就解決了
    * 如果設定都是必要的沒辦法刪除，可以參考保哥的文章 [關於 PATH 環境變數過長而導致命令提示字元下無法執行特定程式的問題](https://blog.miniasp.com/post/2015/09/07/Maximum-length-of-PATH-environment-variable.aspx)

## 心得

檢查環境變數後，發現在安裝 Oracle Developer Tool 過程中額外設定好幾個環境變數，所以造成在安裝 Oracle Developer Tool 後就出現 Visual Studio 無法開啟的問題，結果自己腦補成是 Oracle Developer Tool 造成的 @@"，也因為腦補的關係讓我完全查錯方向多花了很多時間

## 參考資訊

1. [Problems occurred when loading the Microsoft Visual menu](https://stackoverflow.com/questions/25506703/problems-occurred-when-loading-the-microsoft-visual-menu)
2. [關於 PATH 環境變數過長而導致命令提示字元下無法執行特定程式的問題](https://blog.miniasp.com/post/2015/09/07/Maximum-length-of-PATH-environment-variable.aspx)
3. [Visual Studio 怪怪的？！ Visual Studio 好慢？！(Visual Studio 的無痕模式)](http://blog.yowko.com/2016/12/visual-studio-safe-mode.html)
4. [右鍵選單 Create Unit Tests (建立單元測試) 選項不見？！](/2017/05/missing-create-unit-tests.html)
