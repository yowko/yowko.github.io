---
title: "HttpContextBase 找不到 GetOwinContext 定義？！"
date: 2017-11-06T22:15:00+08:00
lastmod: 2021-11-02T22:15:58+08:00
draft: false
tags: ["ASP.NET Identity","ASP.NET MVC","Debug"]
slug: "httpcontextbase-getowincontext"
aliases:
    - /2017/11/httpcontextbase-getowincontext.html
---
## HttpContextBase 找不到 GetOwinContext 定義？

這是在將 ASP.NET Identity 相關功能加至 MVC 專案時所遇到的問題。我在加入 ASP.NET Identity 時習慣使用 Empty 專案範本從無至有加入相關程式碼，而不是由預設 MVC 專案範本開始，主要原因是為了讓專案的程式碼比較乾淨，不會出現不知道該不該刪除的程式碼，而今天遇到的錯誤就是加入 ASP.NET Identity 過程中所遇到的問題

之前曾經分享過如何 [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/add-aspnet-identity-empty-project)，有完整地說明詳細地操作流程，照著做就可以順利完成，但為什麼還有今天的介紹呢？！ 因為人就是一種懶惰的生物，我覺得我應該可以不用看筆記搞定，想不到一下就卡關了XD 加上錯誤訊息讓人很難聯想到需要安裝的 NuGet 套件名稱，為了加深印象特別筆記一篇

## 錯誤訊息

1. 訊息內容

    ```log
    'HttpContextBase' does not contain a definition for 'GetOwinContext' and no extension method 'GetOwinContext' accepting a first argument of type 'HttpContextBase' could be found (are you missing a using directive or an assembly reference?)
    
    Cannot resolve symbol 'GetOwinContext'
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/32445038-7bf349a6-c33f-11e7-8da9-8bc0dd456f46.png)

## 解決方式：安裝 Microsoft.Owin.Host.SystemWeb

* 相依套件
  * Owin (>= 1.0.0)
  * Microsoft.Owin (>= 3.1.0)

* 指令安裝

    > Install-Package Microsoft.Owin.Host.SystemWeb

## 心得

如果依照 [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/add-aspnet-identity-empty-project) 應該不會發生相同錯誤，但明明不久前自己才紀錄過的內容竟然沒幫上自己的忙XD，這就是太 `靠勢` 的下場，再次提醒著自己要時時保持著謙遜跟好奇心，才不會阻擋了自己成長的空間

## 參考資訊

1. ['System.Web.HttpContext' does not contain a definition for 'GetOwinContext'](https://stackoverflow.com/questions/26710434/system-web-httpcontext-does-not-contain-a-definition-for-getowincontext)
2. [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/add-aspnet-identity-empty-project)
