---
title: "ASP.NET MVC 出現 HTTP Error 404.15 - Not Found"
date: 2017-01-13T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC","Debug"]
slug: "aspnet-mvc-http-error-404-15-not-found"
aliases:
    - /2017/01/aspnet-mvc-http-error-404-15-not-found.html
---
## ASP.NET MVC 出現 HTTP Error 404.15 - Not Found

開發 ASP.NET MVC 網站，偶爾會遇到 `HTTP Error 404.15 - Not Found` 的錯誤，"印象中" 造成的原因有好幾個，每次都因為趕著要完成沒有刻意去紀錄，但沒搞清楚難保日後又遇到，就來釐清問題原因吧

## 錯誤訊息及畫面

1. 錯誤訊息 - 英文
    - 訊息內容

        ```log
        HTTP Error 404.15 - Not Found
        The request filtering module is configured to deny a request where the query string is too long.
        
        Most likely causes:
        Request filtering is configured on the Web server to deny the request because the query string is too long.
        
        Things you can try:
        Verify the configuration/system.webServer/security/requestFiltering/requestLimits@maxQueryString setting in the applicationhost.config or web.config file.
        ```

    - 錯誤截圖
        ![1error](https://cloud.githubusercontent.com/assets/3851540/21900376/898d9086-d92e-11e6-8810-1dd637f3132a.png)

2. 錯誤訊息 - 中文
    - 訊息內容

        ```log
        HTTP 錯誤 404.15 - Not Found
        要求篩選模組設定為拒絕查詢過長的要求。
        
        最有可能的原因:
        因為查詢字串過長，網頁伺服器上的要求篩選已設定為拒絕要求。
        
        解決方法:
        確認 applicationhost.config 或 web.config file 檔案中的 configuration/system.webServer/security/requestFiltering/requestLimits@maxQueryString 設定.
        ```

    - 錯誤截圖
        ![2errorzh](https://cloud.githubusercontent.com/assets/3851540/21900377/898df1fc-d92e-11e6-8223-77577f8cace5.png)

## 問題原因

>驗證設定不正確造成頁面一直 redirect

## 可能的解決方式

1. 檢查登入頁的驗證
    - 確認 Action 是允許匿名 `[AllowAnonymous]`

        ![3login](https://cloud.githubusercontent.com/assets/3851540/21900380/89b64e04-d92e-11e6-8180-45a889979ccb.png)

2. 檢查是否允許匿名驗證
    - 開發環境
        1. 專案右鍵

            ![4rightclick](https://cloud.githubusercontent.com/assets/3851540/21900379/898edcc0-d92e-11e6-835f-fec20b1a6034.png)
        2. Anonymous Authentication --> Enable

            ![5enableanonymous](https://cloud.githubusercontent.com/assets/3851540/21900375/898ca05e-d92e-11e6-8c57-2351dc6dd7f9.png)
    - IIS
        1. 網站 --> Authentication

            ![6iisauth](https://cloud.githubusercontent.com/assets/3851540/21900374/898c9334-d92e-11e6-9284-0b317e3de483.png)

        2. Anonymous Authentication --> Enable

            ![7enabliisauth](https://cloud.githubusercontent.com/assets/3851540/21900378/898dfe72-d92e-11e6-9e6f-58701db38c9b.png)

3. 停用 OWIN 啟始探索
    - 在 web.config 加上停用設定

        ```xml
        <add key="owin:AutomaticAppStartup" value="false"/>
        ```

## 參考資料

1. [New Asp.Net MVC5 project produces an infinite loop to login page](http://stackoverflow.com/questions/19601412/new-asp-net-mvc5-project-produces-an-infinite-loop-to-login-page)
