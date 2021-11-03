---
title: "為 ASP.NET WEB API 加上簡易的 Token 驗證"
date: 2017-02-28T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET Web API"]
slug: "aspnet-web-api-fixed-token"
aliases:
    - /2017/02/aspnet-web-api-fixed-token.html
---
## 為 ASP.NET WEB API 加上簡易的 Token 驗證

朋友需要個提供資料處理的 WEB API ，需要有個基本的 TOKEN 來驗證即可，甚至也不需要 TOKEN 的管理功能，直接指定固定 TOKEN 即可，聽來不太合理，但是需求永遠只是低標，就一步步開始吧

## 新增驗證 Handler

1. 專案新增 `Handler` 資料夾
2. 新增 `TokenHandler.cs`
3. `TokenHandler.cs` 繼承 `DelegatingHandler`

    ```cs
    using System.Net.Http;
    ```

4. 定義一個固定 Token

    ```cs
    private const string Token = "3f1472de8cb345f78a826d0cc466bbbe";
    ```

5. 複寫 `SendAsync`

    ```cs
    protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        bool isValid = false;
        IEnumerable<string> headerDatas;
        //嘗試取得 Header 中 "Token" 的資料
        var hasToken = request.Headers.TryGetValues("Token", out headerDatas);
            //request Header 中有 Token
        if (hasToken)
        {
            //檢查 Token 是否與設定的相同
            if (headerDatas?.Any(a=>a.Equals(Token)) == true)
            {
                isValid = true;
            }
        }
            //Token 無效，回傳 401
        if (!isValid)
            return request.CreateResponse(HttpStatusCode.Unauthorized, "Unauthorized");
            //Token 檢查通過，繼續後續作業
        var response = await base.SendAsync(request, cancellationToken);
            //回傳執行結果
        return response;
    }
    ```

## 註冊 `Handler`

- 在 Global.asax.cs 中的 `Application_Start()` 註冊 `Handler`

    >GlobalConfiguration.Configuration.MessageHandlers.Add(new TokenHandler());

    ```cs
    protected void Application_Start()
    {
        略....
        GlobalConfiguration.Configuration.MessageHandlers.Add(new TokenHandler());
    }
    ```

## 使用方式及測試結果

1. Request Header 加入 `Token`

    ```log
    Token:3f1472de8cb345f78a826d0cc466bbbe
    ```

2. Request Header `未` 加入 Token

    ![1fail](https://cloud.githubusercontent.com/assets/3851540/22261295/58adbefa-e2a7-11e6-8447-c0d984920199.png)
3. Request Header 加入 Token

    ![2ok](https://cloud.githubusercontent.com/assets/3851540/22261297/58d94ab6-e2a7-11e6-97fa-8468c4cd41eb.png)

## 心得

很快速地完成 WEB API 固定 Token 的驗證，擴充性實在不足，可以想見很快就需要改寫，但已經滿足基本需求，接著就可以做些程式碼的增強與改善。

## 參考資料

1. [ASP.NET Web API 訊息處理器](http://huan-lin.blogspot.com/2013/01/aspnet-web-api-message-handlers.html)
