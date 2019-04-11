---
title: "怎麼看 ASP.NET Web API 的版本"
date: 2017-10-09T21:34:00+08:00
lastmod: 2018-9-27T21:34:49+08:00
draft: false
tags: ["ASP.NET Web API"]
slug: "aspnet-web-api-version"
aliases:
    - /2017/10/aspnet-web-api-version.html
---
# 怎麼看 ASP.NET Web API 的版本
這幾天在參考 OData 官方文件說明時，留意到內容提到需要使用 ASP.NET Web API 2.2 以上，心裡一陣疑問：現在最新版是 2.3 吧？！ 是文件舊了嗎？ 還是只要 2.2 以上就可以滿足條件？ 立馬打開既有專案看一下用的是哪一版，結果反而更疑惑了XD

## 專案 ASP.NET Web API 版本資訊

自以為的檢查版本方式：

1.  檢查 packages.config

    ![1packageconfig](https://user-images.githubusercontent.com/3851540/31340602-4544e3b8-ad39-11e7-83cc-6a92fd691c0c.png)

    > 原來專案安裝的是 5.2.3 版，這是新版號嗎？！版號規則也差太多了吧，重新確認一下是否為新版號

2.  確認 NuGet 套件版本

    > 透過 NuGet 可以瀏覽到所有版本

    ![2nugerversion](https://user-images.githubusercontent.com/3851540/31340601-4543b542-ad39-11e7-9317-7dc5fee50bc4.png)

    > 看來版號命名規則不是新的XD，那到底要如何辨識 ASP.NET Web API 的版本

## 正確 ASP.NET Web API 版本資訊

透過 NuGet 來看是正確的做法，只是看法錯了，正確的方式如下：

1.  開啟 Microsoft.AspNet.WebApi 在 NuGet 上的說明頁：[Microsoft.AspNet.WebApi](https://www.nuget.org/packages/Microsoft.AspNet.WebApi/)

    ![3api523api22](https://user-images.githubusercontent.com/3851540/31340600-45437c6c-ad39-11e7-8d6f-059dd150f8a5.png)

    > 原來 `Microsoft.AspNet.WebApi 5.2.3` 就是 `Microsoft ASP.NET Web API 2.2`

2.  Microsoft ASP.NET Web API 2 與 Microsoft ASP.NET Web API 1 的分界點呢？

    *   `Microsoft.AspNet.WebApi 5.0.0` 開始是 `Microsoft ASP.NET Web API 2`

        ![4api2](https://user-images.githubusercontent.com/3851540/31340603-45453156-ad39-11e7-8bcf-18c14cdb77ca.png)

    *   `Microsoft.AspNet.WebApi 4.0.30506` 之前是 `Microsoft ASP.NET Web API`

        ![5api](https://user-images.githubusercontent.com/3851540/31340605-45455780-ad39-11e7-949c-41e639275357.png)

    *   `Microsoft.AspNet.WebApi 5.1.0` 則是 `Microsoft ASP.NET Web API 2.1`

        ![6api21](https://user-images.githubusercontent.com/3851540/31340604-45453fac-ad39-11e7-8789-5a66d9fbabac.png)

## 心得

說實話版號的規則並不明確，加上 NuGet 套件名稱 (Microsoft.AspNet.WebApi) 與 ASP.NET Web API 名稱 (Microsoft ASP.NET Web API) 在字的大小寫上都有差異，常常讓人搞不清楚用意為何，不知道有沒有一天可以像是其他工具直接執行 `-v` 就可以知道版本，不過至少 .net core 已經可以了，我想應該未來應該有機會達成才是

# 參考資訊

1.  [How to know the web api project version ?](https://forums.asp.net/t/2102345.aspx?How+to+know+the+web+api+project+version+)
2.  [Microsoft.AspNet.WebApi](https://www.nuget.org/packages/Microsoft.AspNet.WebApi/)
