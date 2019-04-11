---
title: "建立 ASP.NET Web API 專案的幾種方式"
date: 2017-07-23T23:48:00+08:00
lastmod: 2018-09-24T23:48:57+08:00
draft: false
tags: ["ASP.NET Web API"]
slug: "create-aspnet-web-api"
aliases:
    - /2017/07/create-aspnet-web-api.html
---
# 建立 ASP.NET Web API 專案的幾種方式
前端工程與行動裝置的風行，讓問世多年的 ASP.NET Web API 取代 ASP.NET MVC 的態勢愈來愈明顯，網站開發已經由使用單一 framework 處理前後端轉變為由後端只提供資料，畫面顯示部份由前端專責工程師處理或是行動裝置取代

很明顯的改變就是以往建立專案多是使用 ASP.NET MVC ，而近幾年 ASP.NET MVC 已經少見很多，取而代之的都是 ASP.NET Web API。既然 ASP.NET Web API 這麼盛行，也問世好幾年，但一直都沒有好好了解它。剛好最近的專案需要建立多個 ASP.NET Web API 站台，趁這個機會紀錄一下

## 如何建立 ASP.NET Web API 專案

1.  使用 ASP.NET Web API 專案範本

    ![1webapitemplate](https://user-images.githubusercontent.com/3851540/28500732-0c1fdb5e-7000-11e7-96e6-d3eabaf3ba6e.png)

2.  使用 Empty 專案範本 - 加入 Web API 資料夾與參考

    ![2emptywithwebapi](https://user-images.githubusercontent.com/3851540/28500733-0c461e86-7000-11e7-9299-ac0d8a17d7ef.png)

3.  使用 Empty 專案範本

    ![3empty](https://user-images.githubusercontent.com/3851540/28500734-0c614cba-7000-11e7-9bdb-3b7dbc6271b2.png)

## 1. 使用 ASP.NET Web API 專案範本建立 ASP.NET Web API

*   預設專案結構

    ![4webapistructure](https://user-images.githubusercontent.com/3851540/28500735-0c696c38-7000-11e7-8429-e1f53ec32e73.png)

*   預設安裝 `MVC` 與 `Web Api` 功能

    ![5defaultinstall](https://user-images.githubusercontent.com/3851540/28500738-0c6fec34-7000-11e7-8ae4-5aaa6d31976d.png)

    *   因為使用 HelpPage 需要用到 MVC，所以會進行完整安裝

*   優點：

    *   具備 `MVC` 與 `Web Api` 完整功能，可以直接開發及部署

*   缺點：

    *   預設安裝了不一定需要套件(e.g. 不一定需要 HelpPage，可以直接使用 Swagger 取代，導致 MVC 多餘)



## 2. 使用 Empty 專案範本(加入 Web API 資料夾與參考) - 建立 ASP.NET Web API

*   預設專案結構

    ![6emptywebapistructure](https://user-images.githubusercontent.com/3851540/28500737-0c6fd0c8-7000-11e7-8fa6-95f017d288bc.png)

*   預設安裝 `Web Api` 功能

    ![7defaultwebapi](https://user-images.githubusercontent.com/3851540/28500736-0c6daece-7000-11e7-83a7-b63ee461493c.png)

*   優點：


    *   具備 `Web Api` 完整功能，加入 api controller 就可以正常運行

*   缺點：


    *   啟動畫面會是 403.14，對其他使用者較不友善

        ![8403.14](https://user-images.githubusercontent.com/3851540/28500739-0c723642-7000-11e7-8254-1b66bcbf80ae.png)

## 3. 使用 Empty 專案範本建立 ASP.NET Web API

*   預設專案結構

    ![9empty](https://user-images.githubusercontent.com/3851540/28500740-0c90e3bc-7000-11e7-9048-049e3dd8fabb.png)

*   完全沒有預設安裝
*   優點：


    *   非常乾淨，需要什麼完全自訂

*   缺點：


    *   需要較多步驟才能建立可執行的程式

*   內含了兩種建立方法
    *   使用 `Microsoft.AspNet.WebApi`
    *   使用 `Microsoft.AspNet.WebApi.Core`



#### 使用 `Microsoft.AspNet.WebApi` 建立 ASP.NET Web API

1.  NuGet 安裝 `Microsoft.AspNet.WebApi`

    > 2017/07/23 最新版為 v5.2.3

    ![10mswebapi](https://user-images.githubusercontent.com/3851540/28500742-0c96031a-7000-11e7-8015-1258922a5945.png)

2.  加入 `App_Start` 與 `Controllers` 兩個資料夾至專案中

    ![11twofolder](https://user-images.githubusercontent.com/3851540/28500741-0c961c9c-7000-11e7-9b2d-b498cbb27092.png)

3.  在 `App_start` 中加入 `WebApiConfig.cs` 檔

    *   引用 `using System.Web.Http;`

        ```cs
        using System.Web.Http;
        ```

    *   加入註冊 api route 設定

        ```cs
        public static void Register(HttpConfiguration config)
        {
            // Web API routes
            config.MapHttpAttributeRoutes();
                        config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
        ```

    *   完整程式碼

        ```cs
        using System.Web.Http;
        namespace EmptyWebApi.App_Start
        {
            public static  class WebApiConfig
            {
                public static void Register(HttpConfiguration config)
                {
                    // Web API routes
                    config.MapHttpAttributeRoutes();
                                config.Routes.MapHttpRoute(
                        name: "DefaultApi",
                        routeTemplate: "api/{controller}/{id}",
                        defaults: new { id = RouteParameter.Optional }
                    );
                }
            }
        }
        ```

4.  加入 `Global.asax` 檔案

    *   專案 按右鍵 --> Add --> New Item... --> 搜尋 `Global` --> Global Application Class

        ![12global](https://user-images.githubusercontent.com/3851540/28500743-0c9d792e-7000-11e7-8dc6-a7896df5b883.png)

    *   刪除所有內容，只保留 `Application_Start` 並加上 api route 註冊

        ```cs
        protected void Application_Start(object sender, EventArgs e)
        {
            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
        ```

    *   完整程式碼

        ```cs
        using EmptyWebApi.App_Start;
        using System;
        using System.Web.Http;
                    namespace EmptyWebApi
        {
            public class Global : System.Web.HttpApplication
            {
                protected void Application_Start(object sender, EventArgs e)
                {
                    GlobalConfiguration.Configure(WebApiConfig.Register);
                }
                        }
        }
        ```

5.  在 Controllers 資料夾新增 controller 即可

#### 使用 `Microsoft.AspNet.WebApi.Core` 建立 ASP.NET Web API

> 這個方式其實就是使用 Owin 來 host web app

1.  NuGet 安裝 `Microsoft.AspNet.WebApi.Core`

    > 2017/07/23 最新版為 v5.2.3

    ![13emptycore](https://user-images.githubusercontent.com/3851540/28500744-0ca00e46-7000-11e7-8434-b51667210c25.png)

2.  NuGet 安裝 `Microsoft.Owin.Host.SystemWeb`

    > 2017/07/23 最新版為 v3.1.0

    ![14owinsystemweb](https://user-images.githubusercontent.com/3851540/28500745-0ca1f530-7000-11e7-890e-6eed6ee2f66f.png)

3.  NuGet 安裝 `Microsoft.AspNet.WebApi.Owin`

    > 2017/07/23 最新版為 v5.2.3

    ![15owinwebapi](https://user-images.githubusercontent.com/3851540/28500746-0cb949ba-7000-11e7-8b8f-a32b6443a4a5.png)

4.  加入 `App_Start` 與 `Controllers` 兩個資料夾至專案中

    ![11twofolder](https://user-images.githubusercontent.com/3851540/28500741-0c961c9c-7000-11e7-9b2d-b498cbb27092.png)

5.  在 `App_start` 中加入 `WebApiConfig.cs` 檔

    *   引用 `using System.Web.Http;`

        ```cs
        using System.Web.Http;
        ```

    *   加入註冊 api route 設定

        ```cs
        public static void Register(HttpConfiguration config)
        {
            // Web API routes
            config.MapHttpAttributeRoutes();
                        config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
        ```

    *   完整程式碼

        ```cs
        using System.Web.Http;
        namespace EmptyWebApi.App_Start
        {
            public static class WebApiConfig
            {
                public static void Register(HttpConfiguration config)
                {
                    // Web API routes
                    config.MapHttpAttributeRoutes();
                                config.Routes.MapHttpRoute(
                        name: "DefaultApi",
                        routeTemplate: "api/{controller}/{id}",
                        defaults: new { id = RouteParameter.Optional }
                    );
                }
            }
        }
        ```

6.  加入 `Startup.cs` 檔案

    *   專案 按右鍵 --> Add --> New Item... --> Class
    *   加上 api route 註冊

        ```cs
        public void Configuration(IAppBuilder appbuilder)
        {
            var httpConfiguration = new HttpConfiguration();
            WebApiConfig.Register(httpConfiguration);
            appbuilder.UseWebApi(httpConfiguration);
        }
        ```

    *   完整程式碼

        ```cs
        using EmptyWebApi.App_Start;
        using System;
        using System.Web.Http;
                    namespace EmptyWebApi
        {
            public class Startup
            {
                public void Configuration(IAppBuilder appbuilder)
                {
                    var httpConfiguration = new HttpConfiguration();
                    WebApiConfig.Register(httpConfiguration);
                    appbuilder.UseWebApi(httpConfiguration);
                }
            }
        }
        ```

7.  在 `AssemblyInfo.cs` 中加上 Owin 啟動定義

    *   模式

        ```cs
        [assembly: OwinStartup(typeof({ProjectName}.Startup))]
        ```
    *   範例

        ```cs
        [assembly: OwinStartup(typeof(EmptyWebApi.Startup))]
        ```

8.  在 Controllers 資料夾新增 controller 即可

## 心得

建立 ASP.NET Web API 專案最少有以上幾種，尤其透過 Owin 的方式來建立，還可以使用 console 或是其他 host 方式，可以說是各有優缺點，可以依實際需求來選擇適合的建立方式

一般情況下使用 Empty Template with Web Api 是最平衡的，不僅保有快速產生專案架構的好處也不至於安裝太多無用 package

# 參考資訊

1.  [Create Web API project:](http://www.tutorialsteacher.com/webapi/create-web-api-project)
2.  [Web API for .NET Part 1: Creating Your First Project](https://gooroo.io/GoorooTHINK/Article/16421/Web-API-for-NET-Part-1-Creating-Your-First-Project/20100)
3.  [Can't get the OWIN Startup class to run in IIS Express after renaming ASP.NET project file](https://stackoverflow.com/questions/30242581/cant-get-the-owin-startup-class-to-run-in-iis-express-after-renaming-asp-net-pr)
