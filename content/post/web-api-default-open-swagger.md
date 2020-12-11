---
title: "開啟 Web Api Url 時直接導向 Swagger 頁面"
date: 2017-07-24T22:24:00+08:00
lastmod: 2020-12-11T22:24:50+08:00
draft: false
tags: ["套件","ASP.NET Web API"]
slug: "web-api-default-open-swagger"
aliases:
    - /2017/07/web-api-default-open-swagger.html
---
# 開啟 Web Api Url 時直接導向 Swagger 頁面
文章 [建立 ASP.NET Web API 專案的幾種方式](/2017/07/create-aspnet-web-api.html) 介紹到選擇適合的專案範本來建立 ASP.NET Web Api 專案以避免預設安裝過多用不到的套件，其中除了完整安裝的 ASP.NET Web API 專案範本之外，其他安裝方式都不會有 View，所以如果直接使用瀏覽器開啟 Web Api Url 時，會出現 403.14 的錯誤畫面，這對於 user 來說是不友善的，所以針對 dev 或是沒有資安風險的專案，打算預設導向 swagger 畫面，直接提供 user 相關 api 文件及使用方式，也可以有效隱藏 錯誤畫面

來看看怎麼設定吧

## 預設啟動畫面

啟動畫面會是 403.14，對其他使用者較不友善

![8403.14](https://user-images.githubusercontent.com/3851540/28500739-0c723642-7000-11e7-8254-1b66bcbf80ae.png)

## 關於 Swashbuckle

Swashbuckle 是 Swagger 在 .NET 上的實做版本，可以提供 web api 的說明文件，也可以直接在畫面上發出 request，功能上就如同 ASP.NET Web Api HelpPage 與 PostMan 的整合體，功能很強大，非常方便

安裝方式：

1.  NuGet 搜尋 `Swashbuckle` or `Swagger`

    ![1nuget](https://user-images.githubusercontent.com/3851540/28527626-adeb20c0-70bd-11e7-96d4-9204aff7f6bf.png)

2.  啟用專案的 XML 文件

    *   專案 按右鍵 --> Properties

    *   Build --> 勾選 `XML documentation file`

        > 記得將 xml 路徑記起來，待會兒設定會用到

        ![2enablexmldoc](https://user-images.githubusercontent.com/3851540/28527623-ade08746-70bd-11e7-8ca2-230af97c5f24.png)

        
3.  啟用 Swashbuckle (Swagger)

    *   打開 `App_Start` 資料夾下的 `SwaggerConfig.cs`
    *   移除 104 行 (`c.IncludeXmlComments(GetXmlCommentsPath());`) 的註解
    *   新增 `GetXmlCommentPath()` function 並回傳剛剛設定的 XML 路徑

        ```cs
        protected static string GetXmlCommentsPath()
        {
            return  $"{System.AppDomain.CurrentDomain.BaseDirectory}\\bin\\EmptyWebApi-Core.xml";
        }
        ```

4.  直接在專案網址根路徑加上 `/swagger` 即可開啟

    ![3result](https://user-images.githubusercontent.com/3851540/28527622-adde4ec2-70bd-11e7-8d42-8ffa34b14a8a.png)

## 設定預設導向 Swashbuckle 頁面

1.  在 `RouteConfig` 中加入一組 route rule
2.  指定 route rule 使用 `Swashbuckle.Application.RedirectHandler`

    ```cs
    config.routes.MapHttpRoute(
        name: "swagger_root",
        routeTemplate: "",
        defaults: null,
        constraints: null,
        handler: new Swashbuckle.Application.RedirectHandler((message => message.RequestUri.ToString()), "swagger")
    );
    ```

    *   route template 完全沒有 request 參數時導向 `\swagger`
    *   想了解如何轉導，請參考 [RedirectHandler.cs](https://github.com/domaindrivendev/Swashbuckle/blob/e0053e1864defa3c4f73ca2a960eb876e257cc9e/Swashbuckle.Core/Application/RedirectHandler.cs)

3.  Redirect

    ![4redirect](https://user-images.githubusercontent.com/3851540/28527624-ade2c2a4-70bd-11e7-8b4f-e756cfe69ca2.png)

4.  優化

    *   從上面的 redirect 流程來看，我們可以發現 `/` 會 redirect 至 `/swagger` 再 redirect 至 `/swagger/ui/index`
    *   我們可以直接從 `/` redirect 至 `/swagger/ui/index` 以減少一次 redirect

        ```cs
        config.Routes.MapHttpRoute(
            name: "swagger_root",
            routeTemplate: "",
            defaults: null,
            constraints: null,
            handler: new Swashbuckle.Application.RedirectHandler((message => message.RequestUri.ToString()), "swagger/ui/index")
        );
        ```

        ![5lessredirect](https://user-images.githubusercontent.com/3851540/28527625-ade44372-70bd-11e7-81bb-55c2e65d8b5d.png)

## 心得

小小的修改，省了每次打 swagger 的時間，節省的幅度並不高，不過開啟 url 時直接導向的使用者體驗比起之前出現錯誤畫面提昇不少

需要注意的是，專案上線至 production 時要留意安全性，避免對外透露過多資訊，讓有心人士有攻擊的機會跟依據

# 參考資訊

1.  [How to use Swagger as Welcome Page of IAppBuilder in WebAPI](https://stackoverflow.com/questions/30028736/how-to-use-swagger-as-welcome-page-of-iappbuilder-in-webapi)
2.  [RedirectHandler.cs](https://github.com/domaindrivendev/Swashbuckle/blob/e0053e1864defa3c4f73ca2a960eb876e257cc9e/Swashbuckle.Core/Application/RedirectHandler.cs)
