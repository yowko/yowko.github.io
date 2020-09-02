---
title: "將 IIS 站台離線做法 - 使用 App_offline.htm"
date: 2017-10-27T00:45:00+08:00
lastmod: 2020-09-01T00:45:41+08:00
draft: false
tags: ["IIS"]
slug: "iis-app-offline"
aliases:
    - /2017/10/iis-app-offline.html
---
# 將 IIS 站台離線做法 - 使用 App_offline.htm
同事問到 `App_offline.htm` 的功用，一時之間沒意會過來，只覺得應該沒用過，直到同事提到是用來將 iis 站台下線用，才想起原來是它！！ 既然忘記當然就要紀錄一下囉

## `App_offline.htm` 簡介

*   功能

    > 用來將網站離線，可自訂訊息內容提供存取網站的使用者充份資訊，以改善使用者體驗，不致於看到錯誤畫面

*   支援情況

    > 從 ASP.NET 2.0 開始支援，經測試 IIS 6.0 至 IIS 10.0 都可以正常使用，ASP.NET MVC、ASP.NET Core 皆可正常支援

## 使用方式

1.  在網站根目錄中建立 `App_offline.htm`

    > `App_offline.htm` 內容以 html 語法編輯

2.  更新網站內容
3.  移除 `App_offline.htm`


## 實際使用

1.  MVC 站台
    *   正常服務

        ![1mvcnormal](https://user-images.githubusercontent.com/3851540/32065529-92b0dbec-baaf-11e7-9d97-bd8e579ba29e.png)

    *   加上 `App_offline.htm`

        ![2mvcoffline](https://user-images.githubusercontent.com/3851540/32065531-92ff4e26-baaf-11e7-99cf-b96a67b688f9.png)

2.  靜態網頁站台
    *   正常服務

        ![3htmlnoramal](https://user-images.githubusercontent.com/3851540/32065533-93412526-baaf-11e7-88b7-9995e97e48ce.png)

    *   加上 `App_offline.htm`

        ![4htmloffline](https://user-images.githubusercontent.com/3851540/32065535-9373bbda-baaf-11e7-9741-c04ce94169d2.png)

    *   仍然可以直接存取特定網頁

        ![5htmlpass](https://user-images.githubusercontent.com/3851540/32065536-939e5a20-baaf-11e7-8cf4-d8754b576ff8.png)

## 心得

`App_offline.htm` 用法如果不是經同事一問，我想可能會一直遺忘下去，剛好趁這個機會順便測試了 IIS 8 以上版本，確認一樣可行，只是有點美中不足的是在靜態網頁站台，如果直接使用特定頁面存取可以 bypass `App_offline.htm` 的設定，這點要特別留意

# 參考資訊

1.  [逐步解說：使用 XCOPY 部署 ASP.NET Web 應用程式](https://msdn.microsoft.com/zh-tw/library/f735abw9%28v=vs.100%29.aspx)
2.  [HOW TO：準備部署 Web 專案](https://msdn.microsoft.com/zh-tw/library/ff925031%28v=vs.100%29.aspx)
3.  [ASP.NET 核心模組的組態參考](https://docs.microsoft.com/zh-tw/aspnet/core/hosting/aspnet-core-module?WT.mc_id=DOP-MVP-5002594)
