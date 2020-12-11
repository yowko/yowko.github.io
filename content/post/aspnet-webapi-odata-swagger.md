---
title: "設定 ASP.NET WebAPI OData 的 Swagger"
date: 2017-10-15T17:45:00+08:00
lastmod: 2020-12-11T17:45:17+08:00
draft: false
tags: ["套件","ASP.NET Web API","OData"]
slug: "aspnet-webapi-odata-swagger"
aliases:
    - /2017/10/aspnet-webapi-odata-swagger.html
---
# 設定 ASP.NET WebAPI OData 的 Swagger
Swagger 的強大功能不需要再重複強調，重要性在團隊開發上已經扮演著不可或缺的地位，常常在建立 Web API 專案後就會順手將 swagger 設定好，而在以 Web API 為基礎的 OData，當然也可以加上 swagger ，就來看看該如何設定吧

*   以下設定相容使用 `OData V4`
  

## 安裝 NuGet 套件

該套件僅支援 OData V4，如果還在使用 OData V3 可以考慮升級 OData v4，詳細做法請參考 [如何將 OData v3 升級為 OData v4](/2017/10/odata-v3-to-odata-v4.html)

```
Install-Package Swashbuckle.OData
```

![0install](https://user-images.githubusercontent.com/3851540/31583489-ae5d909e-b1cf-11e7-91c1-80c170edf89a.png)

## 設定 SwaggerConfig

1.  在 `App_Start` 資料夾加上 `SwaggerConfig.cs`
2.  在 `SwaggerConfig.cs` 加上 `Register()`

    ```cs
    public static void Register()
    {
        GlobalConfiguration.Configuration
            .EnableSwagger(c =>
            {
                c.SingleApiVersion("v1", "TestOData3");//這個設定請依實際情境調整
                c.CustomProvider(defaultProvider => new ODataSwaggerProvider(defaultProvider, c, GlobalConfiguration.Configuration));
            })
            .EnableSwaggerUi();
    }
    ```

3.  在 namespace 上加入註冊 attribute

    ```cs
    [assembly: PreApplicationStartMethod(typeof(SwaggerConfig), "Register")]
    ```

4.  完整 `SwaggerConfig.cs` 內容

    ```cs
    using System.Web;
    using Swashbuckle.Application;
    using Swashbuckle.OData;
    using System.Web.Http;
    using TestOData3.App_Start;
    
    [assembly: PreApplicationStartMethod(typeof(SwaggerConfig), "Register")]
    namespace TestOData3.App_Start
    {
        public class SwaggerConfig
        {
            public static void Register()
            {
                GlobalConfiguration.Configuration
                    .EnableSwagger(c =>
                    {
                        c.SingleApiVersion("v1", "TestOData3");//這個設定請依實際情境調整
                        c.CustomProvider(defaultProvider => new ODataSwaggerProvider(defaultProvider, c, GlobalConfiguration.Configuration));
                    })
                    .EnableSwaggerUi();
            }
        }
    }
    ```

## 設定 Route

1.  在 `App_Start` 資料夾加上 `RouteConfig.cs`
2.  在 `RouteConfig.cs` 加上 `RegisterRoutes(RouteCollection routes)`

    ```cs
    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.MapHttpRoute(
            name: "swagger_root",
            routeTemplate: "",
            defaults: null,
            constraints: null,
            handler: new RedirectHandler((message => message.RequestUri.ToString()), "swagger")
        );
    }
    ```

3.  完整 `RouteConfig.cs` 內容

    ```cs
    using System.Web.Http;
    using System.Web.Routing;
    using Swashbuckle.Application;
                
    namespace TestOData3.App_Start
    {
        public class RouteConfig
        {
            public static void RegisterRoutes(RouteCollection routes)
            {
                routes.MapHttpRoute(
                    name: "swagger_root",
                    routeTemplate: "",
                    defaults: null,
                    constraints: null,
                    handler: new RedirectHandler((message => message.RequestUri.ToString()), "swagger")
                );
            }
        }
    }
    ```

4.  將 RouteConfig 註冊至 `Global.asax`

    ```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            GlobalConfiguration.Configure(WebApiConfig.Register);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
        }
    }
    ```

## 顯示自訂訊息

1.  開啟專案 XML 註解並設定 XML 位置

    ![1xmldoc](https://user-images.githubusercontent.com/3851540/31583490-ae8e680e-b1cf-11e7-8ca7-575a6704815c.png)

2.  在 `SwaggerConfig.cs` 設定讀取 XML

    *   加上以下設定

        ```cs
        var baseDirectory = AppDomain.CurrentDomain.BaseDirectory;
        var commentsFileName = Assembly.GetExecutingAssembly().GetName().Name + ".XML";
        var commentsFile = Path.Combine(baseDirectory,"bin", commentsFileName);//檔案位置請與第一步 xml 位置相同
        c.IncludeXmlComments(commentsFile);
        ```

    *   完整內容

        ```cs
        using System.Web;
        using Swashbuckle.Application;
        using Swashbuckle.OData;
        using System.Web.Http;
        using TestOData3.App_Start;
        using System;
        using System.IO;
        using System.Reflection;
                    
        [assembly: PreApplicationStartMethod(typeof(SwaggerConfig), "Register")]
        namespace TestOData3.App_Start
        {
            public class SwaggerConfig
            {
                public static void Register()
                {
                    GlobalConfiguration.Configuration
                        .EnableSwagger(c =>
                        {
                            c.SingleApiVersion("v1", "TestOData3");//這個設定請依實際情境調整
                            c.CustomProvider(defaultProvider => new ODataSwaggerProvider(defaultProvider, c, GlobalConfiguration.Configuration));
                            var baseDirectory = AppDomain.CurrentDomain.BaseDirectory;
                            var commentsFileName = Assembly.GetExecutingAssembly().GetName().Name + ".XML";
                            var commentsFile = Path.Combine(baseDirectory,"bin", commentsFileName);//檔案位置請與第一步 xml 位置相同
                            c.IncludeXmlComments(commentsFile);
                        })
                        .EnableSwaggerUi();
                }
            }
        }
        ```

3.  記得在要顯示自訂說明的方法上加上 xml 註解


## 實際效果

在網址根目錄加上 `/swagger` 即可開啟 (ex：`http://localhost:14264/swagger` 會直動導向 `http://localhost:14264/swagger/ui/index`)

![2result](https://user-images.githubusercontent.com/3851540/31583491-aebabc1a-b1cf-11e7-8d33-f0d6c15c24dd.png)

## 心得

這又是一個設定時覺得難度不高，但重新設定又困難重重的例子，我想主要是文件的官方說明不是那麼完整，設定時需要東找西找，加上每個專案狀況不一，可能遇到的問題不太一樣所造成的，經過這次紀錄我想下次花的時間一定會少很多的

# 參考資訊

1.  [如何將 OData v3 升級為 OData v4](/2017/10/odata-v3-to-odata-v4.html)
2.  [rbeauchamp/Swashbuckle.OData](https://github.com/rbeauchamp/Swashbuckle.OData)
