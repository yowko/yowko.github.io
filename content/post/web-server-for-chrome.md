---
title: "透過 Chrome Extension 將靜態網頁開啟成網站"
date: 2017-09-04T21:00:00+08:00
lastmod: 2018-09-25T21:00:11+08:00
draft: false
tags: ["套件","Tools"]
slug: "web-server-for-chrome"
aliases:
    - /2017/09/web-server-for-chrome.html
---
# 透過 Chrome Extension 將靜態網頁開啟成網站
這個好用的 Chrome 套件是在參加 Kuro 大的 Vue.js 課程時，由 Kuro 介紹的開發輔助工具，看到 demo 時驚為天人 也太好用了吧，所以特別筆記一下

以往收到網頁靜態檔，大多都是切版完成的成品，雖然不一定需要部署成網站，但以網站的型式開啟還是可以幫助了解整個畫面流程及有利組件拆分規劃，在這之前我一直都是使用 WebMatrix 的功能，它也可以將靜態網頁以網站方式開啟

## WebMatrix 的用法

1.  將靜態以網站方式開啟

    > File --> Open --> Folder as Site

    ![1openassite](https://user-images.githubusercontent.com/3851540/30013529-bb0f100e-9179-11e7-8485-e516f586f1e2.png)

2.  將網頁使用瀏覽器開啟

    > 在想要的網頁(index.html)上 按右鍵 --> Launch in browser

    ![2launch](https://user-images.githubusercontent.com/3851540/30013531-bb138972-9179-11e7-8e04-e7d8bc812b04.png)

3.  在瀏覽器中使用網頁

    > 會自動指定 port

    ![3siteview](https://user-images.githubusercontent.com/3851540/30013537-bb834776-9179-11e7-8e25-4a82ff66e09d.png)

## Web Server for Chrome 的用法

1.  在 Chrome App Store 針對 `應用程式` 搜尋 `web server`，將 `Web Server for Chrome` 加到 Chrome

    ![4addtochrome](https://user-images.githubusercontent.com/3851540/30013532-bb14233c-9179-11e7-94e0-ca3d73b83e9d.png)

2.  在 Chrome Apps 按下 `Web Server`

    ![5apps](https://user-images.githubusercontent.com/3851540/30013530-bb136636-9179-11e7-8a33-16b04e6b9a8f.png)

3.  選擇網頁資料夾

    ![6choosefolder](https://user-images.githubusercontent.com/3851540/30013533-bb19a780-9179-11e7-9b82-32e6a50ee744.png)

4.  啟動 Web Server，啟動後會出現連線 url

    ![7url](https://user-images.githubusercontent.com/3851540/30013534-bb34277c-9179-11e7-8f90-13fe6230b2a1.png)

5.  選擇想要開啟的頁面

    ![8page](https://user-images.githubusercontent.com/3851540/30013536-bb3af53e-9179-11e7-8b93-b525a54a588b.png)

6.  開啟網頁

    ![9page](https://user-images.githubusercontent.com/3851540/30013535-bb3ae08a-9179-11e7-8b14-a284abe3e1d6.png)

## 心得

比起 WebMatrix 輕量許多，單純以開啟靜態頁面的需求而已，使用 Web Server for Chrome 就足夠了，設定很簡潔、操作上也很簡便，非常優優秀的小工具，可以列入開發必備工具清單之中了

# 參考資訊

1.  [Chrome Web Server - an HTTP web server for Chrome (chrome.sockets)](https://github.com/kzahel/web-server-chrome)
