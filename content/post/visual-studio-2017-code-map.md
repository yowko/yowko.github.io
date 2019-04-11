---
title: "Visual Studio 2017 如何使用 Code Map 功能"
date: 2017-04-11T01:00:00+08:00
lastmod: 2018-09-16T01:00:21+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-2017-code-map"
aliases:
    - /2017/04/visual-studio-2017-code-map.html
---
# Visual Studio 2017 如何使用 Code Map 功能
自從 Visual Studio 2012 加入 Code Map 功能後，在面對前人遺留的 legend code 與複雜程式碼相依參考的問題時不再每每都有離職的衝動，雖然問題沒有被直接解決，但 Code Map 確實大幅降低問題處理的複雜度，讓我們可以透過圖形化方式來幫助理解專案的關連性，關於 Code Map 的詳細介紹可以參考 [快速理解一堆程式碼的方法 – 使用 Visual Studio 2013 的 Code Map 程式碼地圖](https://blogs.msdn.microsoft.com/msdntaiwan/2014/06/29/visual-studio-2013-code-map/) 與 [再論 Code Map – 在 Visual Studio 中進行圖形化偵錯，找 Bug 更容易](https://blogs.msdn.microsoft.com/msdntaiwan/2014/06/29/code-map-visual-studio-bug/)

最近接手一個歷史專案，耳聞一個 controller 有上萬行程式碼XD，打算用 Visual Studio 2017 generate Code Map，第一時間沒找到相關功能，筆記一下讓其他人參考

找尋 Code Map 的過程中發現，安裝程式中有關於 Code Map 功能選單，猜測是 Visual Studio 2017 安裝體驗改善的一部份，原本在 enterprise 預設安裝的 Code Map 功能 改為可選式安裝，讓不需要該功能的使用者可以不安裝以縮短安裝時間

## 如何安裝 Code Map

1.  重新執行 Visual Studio 2017 安裝檔 --> Modify
    
    ![1update](https://cloud.githubusercontent.com/assets/3851540/24850307/1d226250-1e02-11e7-8330-0d1bfe33964b.png)

2.  Individual components --> Code tools --> 勾選 `Code Map`(會自動選取 `DGML editor`) --> Modify

    ![2codemap](https://cloud.githubusercontent.com/assets/3851540/24850309/1d26a54a-1e02-11e7-8f98-1bc33d67d5ec.png)

## 安裝前後比較

*   安裝前
    
    ![3before](https://cloud.githubusercontent.com/assets/3851540/24850308/1d255186-1e02-11e7-8eae-39d0fea1a44e.png)

*   安裝後

    ![4after](https://cloud.githubusercontent.com/assets/3851540/24850310/1d2a2aee-1e02-11e7-8c42-9c0ed29f0967.png)

# 參考資訊

1.  [快速理解一堆程式碼的方法 – 使用 Visual Studio 2013 的 Code Map 程式碼地圖](https://blogs.msdn.microsoft.com/msdntaiwan/2014/06/29/visual-studio-2013-code-map/)
2.  [再論 Code Map – 在 Visual Studio 中進行圖形化偵錯，找 Bug 更容易](https://blogs.msdn.microsoft.com/msdntaiwan/2014/06/29/code-map-visual-studio-bug/)
