---
title: "ASP.NET Core WebAPI 回應 406 Not Acceptable"
date: 2019-06-30T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","ASP.NET WebAPI"]
slug: "aspdotnetcore-webapi-406-not-acceptable"
---

## ASP.NET Core WebAPI 回應 406 Not Acceptable

照著之前筆記 [從 Empty 建立 ASP.NET Core Web API](/add-webapi-to-aspdotnetcore-empty/) 從空專案開始建立 ASP.NET Core WebAPI ，過程中一切順利直到開始加入商業邏輯時卻出現意料外的錯誤，雖然事後覺得我應該要馬上聯想的問題原因，但事實上並沒有，紀錄一下加深印象

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet package

    - Microsoft.AspNetCore.Mvc.FormattersJson 2.2.0

## 錯誤訊息

1. 訊息內容

    ```txt
    406 Not Acceptable
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/60398221-7908fb00-9b88-11e9-87e7-7daf555b9b62.png)

## 問題發生原因

1. 先檢視 kestrel log 的異常

    ```txt
    warn: Microsoft.AspNetCore.Mvc.Infrastructure.DefaultOutputFormatterSelector[1]
      No output formatter was found for content type '' to write the response.
    warn: Microsoft.AspNetCore.Mvc.Infrastructure.ObjectResultExecutor[1]
      No output formatter was found for content type '' to write the response.
    ```

    ![2kestrel](https://user-images.githubusercontent.com/3851540/60398222-7908fb00-9b88-11e9-8db9-92030dcd45a2.png)

2. 發現問題

    從 log 中我們可以看得出來，在 response 時並沒有任何的 content-type，而這個問題的源頭其實可以從 [ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別](/aspdotnet-core-addmvc-addmvccore/) 中看得出來，因為使用 `AddMvcCore()` 而少了 `builder.AddJsonFormatters();`

## 如何解決問題

1. 安裝需要的 formatter

    > 以 json 為例 `Microsoft.AspNetCore.Mvc.FormattersJson` 

    - Package Manager

        ```txt
        Install-Package Microsoft.AspNetCore.Mvc.Formatters.Json
        ```

    - .NET CLI

        ```bash
        dotnet add package Microsoft.AspNetCore.Mvc.Formatters.Json --version 2.2.0
        ```

2. 註冊 formatter

    調整 `Startup.cs` 中的 `ConfigureServices` 方法

    ```cs
     public void ConfigureServices(IServiceCollection services)
    {
            services.AddMvcCore().AddJsonFormatters();
    }
    ```

## 心得

原本以為我看過資料，遇到相關問題我應該可以快速想到問題發生的原因，想不到還是卡了一下XD  

不過錯誤都已經發生了還是要想辦法改善，後來仔細檢討後發現，我之前 trace code 時是直接透過 GitHub 的 search in repository，結果就找到 master 的 code，而我應該要換到 tag `2.2.5` 才是我正使用的版本呀  真是糟糕@@"  幸虧有筆記  才發現之前的錯誤

## 參考資訊

1. [在 ASP.NET Core Web API 中格式化回應資料](https://docs.microsoft.com/zh-tw/aspnet/core/web-api/advanced/formatting?WT.mc_id=DOP-MVP-5002594)
2. [The difference between AddMvc() and AddMvcCore()](https://offering.solutions/blog/articles/2017/02/07/the-difference-between-addmvc-and-addmvccore/)
3. [從 Empty 建立 ASP.NET Core Web API](/add-webapi-to-aspdotnetcore-empty/)
4. [ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別](/aspdotnet-core-addmvc-addmvccore/)
