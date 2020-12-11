---
title: "從 Empty 建立 ASP.NET Core Web API"
date: 2019-06-29T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["ASP.NET Core"]
slug: "add-webapi-to-aspdotnetcore-empty"
---

# 從 Empty 建立 ASP.NET Core Web API

之前曾經在筆記 [建立ASP.NET Web API 專案的幾種方式- Yowko's Notes](/create-aspnet-web-api/) 提到專案的起源分為兩派：

1. 使用 Empty 專案範本再手動安裝需要的 framework
2. 直接使用需要 framework 的專案範本

身為一個懶惰工程師，我就是挑當下允許最方便的方式開始，不過這只限於自己的專案，如果是工作上專案我還是覺得應該從 Empty 開始比較正確，避免預設範本裝了一些多餘的東西進去，造成日後維護員的困擾

## 基本環境脫明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet Package

    - Microsoft.AspNetCore.Mvc.Core 2.2.5

## 如何修改

1. 建立 Empty project

    ```bash
    dotnet new web
    ```

2. 安裝 NuGet 套件：`Microsoft.AspNetCore.Mvc.Core`

    - Package Manager

        ```
        Install-Package Microsoft.AspNetCore.Mvc.Core
        ```

    - .NET CLI

        ```bash
        dotnet add package Microsoft.AspNetCore.Mvc.Core
        ```

3. 註冊 `AddMvcCore()`

    在 `Startup.cs` 中的 `ConfigureServices` 方法註冊 `AddMvcCore()`

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvcCore();
    }
    ```

    > 該用 `AddMvc()` or `AddMvcCore()` 可以參考之前筆記 [ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別](/aspdotnet-core-addmvc-addmvccore)

4. 註冊 WebAPI Route

    - 使用預設路由

        在 `Startup.cs` 中的 `Configure` 方法註冊所需的 route

        ```cs
        app.UseMvc(routes =>
        {
            routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
        });
        ```

    - 使用 Route attribute

        - 在 `Startup.cs` 中的 `Configure` 方法加入 mvc

            ```cs
            app.UseMvc();
            ```

        - 在目前 Controller 上加上 route attribute 及 ApiController

            ```cs
            [Route("[controller]/[action]")]
            [ApiController]
            ```

5. 新增 Controller 繼承自 `ControllerBase`

    > MVC 繼承自 `Controller` 而 Web API 只需繼承 `ControllerBase`，兩者差異可以參考之前筆記 [ASP.NET Core 中 Controller 與 ControllerBase 的差別](/aspdotnet-core-controller-controllerbase/)

    ```cs
    using Microsoft.AspNetCore.Mvc;

    namespace AddWebApiFromEmpty.Controllers
    {
        [Route("[controller]/[action]")]
        [ApiController]
        public class ValuesController : ControllerBase
        {
            // GET
            public IActionResult Hello()
            {
                return Content("Hello WebApi");
            }
        }
    }
    ```

## 心得

.NET Core 愈用愈覺得自己不知道的東西好多，有時既有功能都還沒摸懂就又來新的特性，行為又常常跟過去認識的不太一樣XD，不時覺得功能特性試不出來有一部份是被過去經驗誤導，所幸 .NET Core 的文件很清楚

另外透過筆記的方式一步步釐清不同的東西實在是個不錯方法，反正不懂的就多看、看懂了就多記，加深一下印象，相信有天應該可以比較上手的

## 參考資訊

1. [ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別](/aspdotnet-core-addmvc-addmvccore)
2. [Create a minimal ASP.NET Core 2.0 MVC web application](https://www.iambacon.co.uk/blog/create-a-minimal-asp-net-core-2-0-mvc-web-application)
3. [ASP.NET Core 中 Controller 與 ControllerBase 的差別](/aspdotnet-core-controller-controllerbase/)
4. [使用 ASP.NET Core 建立 Web API](https://docs.microsoft.com/zh-tw/aspnet/core/web-api/index?WT.mc_id=DOP-MVP-5002594)
5. [教學課程：使用 ASP.NET Core MVC 建立 Web API](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/first-web-api?WT.mc_id=DOP-MVP-5002594&tabs=visual-studio#add-a-controller)
6. [ASP.NET Core 中的路由](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/routing?WT.mc_id=DOP-MVP-5002594)
