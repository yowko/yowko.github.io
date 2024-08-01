---
title: "ASP.NET 8 新增的錯誤處理"
date: 2024-07-31T00:30:00+08:00
lastmod: 2024-07-31T00:30:31+08:00
draft: false
tags: ["csharp","aspdotnet"]
slug: "aspnetcore-8-error-handling"
---

## ASP.NET 8 新增的錯誤處理

ASP.NET Core 問市時提供了 Middleware 的做法：可以透過建立不同元件用做 pipeline 處理請求，讓 log、authentication、authorization、error handling 等功能可以將處理邏輯分離，不用與商業邏輯混在一起，其中 error handling 在使用 middleware 方式實作全域 exception handling 後不僅有更高的彈性，程式碼也更加簡潔

最近團隊專案陸續升級到 ASP.NET Core 8，發現官網在 error handling 的方式有不同做法，所以今天就來體驗看看新的方式

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack 1.6.3 (17138)
- .NET SDK 8.0.101
- JetBrains Rider 2024.1.4

## ASP.NET 的錯誤處理

1. 使用 middleware (傳統做法；ASP.NET 8 也能用)

    - 建立 middleware 來處理錯誤

        {{<gist yowko 992366119cd9920c15742aa5c76a4f1b "ExceptionMiddleware.cs">}}

    - 在 Programs.cs 使用 middleware

        > 加上 `app.UseMiddleware<ExceptionMiddleware>();`

        {{<gist yowko 992366119cd9920c15742aa5c76a4f1b "MiddlewareProgram.cs">}}

    - 實際效果

        ![1middleware](https://github.com/user-attachments/assets/5e08831f-9e08-4748-9a0f-8a0db1a29c53)

        ![2middleware](https://github.com/user-attachments/assets/dc0ff025-ef96-474c-a4a6-e4d60acf6dda)

2. 使用 Problem details (我看文件從 ASP.NET 2.1 開始就有 @@")

    - 在註冊呼叫 `AddProblemDetails` 以下的 middleware 會連帶產生 ProblemDetails 的 http response

        - ExceptionHandlerMiddleware：會在未定義自訂處理常式時產生問題詳細資料回應。
        - StatusCodePagesMiddleware：預設會產生問題詳細資料回應。
        - DeveloperExceptionPageMiddleware：在 Accept 要求 HTTP 標頭不包含 text/html 時，在開發過程中產生問題詳細資料回應。

    - 在 Programs.cs 註冊使用

        > 加上 `builder.Services.AddProblemDetails();` 與 `app.UseExceptionHandler();`

        {{<gist yowko 992366119cd9920c15742aa5c76a4f1b "ProblemDetailProgram.cs">}}

    - 實際效果

        ![3problemdetail](https://github.com/user-attachments/assets/e640fcb9-b557-4e3e-ba51-5a3533e8102d)

        ![4problemdetail](https://github.com/user-attachments/assets/c5b9f8b4-b3b6-47ac-9da3-ce9a008476f6)

3. 使用 IExceptionHandler (ASP.NET 8 新增)

    - 建立 ExceptionHandler 來處理錯誤

        {{<gist yowko 992366119cd9920c15742aa5c76a4f1b "GlobalExceptionHandle.cs">}}

    - 在 Programs.cs 註冊使用

        > 加上 `builder.Services.AddExceptionHandler<GlobalExceptionHandle>();` 與 `app.UseExceptionHandler("/Error");`

        {{<gist yowko 992366119cd9920c15742aa5c76a4f1b "IExceptionHandlerpProgram.cs">}}

    - 實際效果

        ![5iexceptionhandler](https://github.com/user-attachments/assets/19519ff0-21b0-4e2c-b8bc-96bc4e0f856c)

        ![6iexceptionhandler](https://github.com/user-attachments/assets/e9de6e29-d1ce-494f-8c90-b1446d16f7d0)

## 心得

1. IExceptionHandler 的確是更加符合意圖，但以程式碼來看我個人覺得 middleware 為簡潔 (一行 `app.UseMiddleware` 就搞定)，初步看來應該是為了避開 exception 的昂貴成本
2. IExceptionHandler 有其彈性跟配套：IExceptionHandler 回傳 true/false 來判斷是否處理錯誤，並且可以透過 `app.UseExceptionHandler("/Error");` 來指定處理錯誤的路由，但目前還沒想到實際應用的場景
3. IExceptionHandler 目前還沒辦法單獨存在，一定會搭配 `Microsoft.AspNetCore.Diagnostics.ExceptionHandlerMiddleware` 造成 log 會是成對出現，目前僅能透過 config 關閉 Microsoft.AspNetCore.Diagnostics.ExceptionHandlerMiddleware 的 log 輸出
    {{<gist yowko 992366119cd9920c15742aa5c76a4f1b "appsettings.json">}}

## 參考資訊

1. [Microsoft Learn:Handle errors in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594)
2. [Why does ASP.NET Core log an unhandled exception when using a global exception handler?](https://learn.microsoft.com/en-us/answers/questions/1618819/why-does-asp-net-core-log-an-unhandled-exception-w?WT.mc_id=DOP-MVP-5002594)
3. [Global Exception Handling in ASP.NET Core - IExceptionHandler in .NET 8 [Recommended]](https://codewithmukesh.com/blog/global-exception-handling-in-aspnet-core/)
4. [Handling Errors with IExceptionHandler in ASP.NET Core 8.0](https://medium.com/@AntonAntonov88/handling-errors-with-iexceptionhandler-in-asp-net-core-8-0-48c71654cc2e)
5. [Global Error Handling in ASP.NET Core 8](https://www.milanjovanovic.tech/blog/global-error-handling-in-aspnetcore-8)
6. [The new way of Error Handling in .Net 8](https://bugrasitemkar.medium.com/the-new-way-of-error-handling-in-net-8-176c8772520e)
