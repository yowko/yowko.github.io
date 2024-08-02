---
title: "ASP.NET ProblemDetail 錯誤處理"
date: 2024-08-01T00:30:00+08:00
lastmod: 2024-08-01T00:30:31+08:00
draft: false
tags: ["csharp","aspdotnet"]
slug: "aspnetcore-problemdetail-error-handling"
---

## ASP.NET ProblemDetail 錯誤處理

之前筆記 [ASP.NET 8 新增的錯誤處理](/aspnetcore-8-error-handling/) 主要是想嘗試 ASP.NET 8 新增的錯誤處理功能，不過剛好官網文件 [Microsoft Learn:Handle errors in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594) 有提到其他錯誤處理做法，所以一併簡單紀錄，只是官網上的範例大多數情況下沒辦法滿足實際需求，所以今天就是進一步了解 ASP.NET 8 的 `ProblemDetail` 錯誤處理。

ProblemDetails 是 [IETF RFC 7807](https://tools.ietf.org/html/rfc7807) 在 ASP.NET 中的實作，主要是為了標準化異常回應，讓 client 能夠更容易理解錯誤訊息

從 ASP.NET Core 2.2 開始，在 controller 加上 `[ApiController]` 並使用 ControllerBase 內建方法返回 HTTP 狀態代碼回應（如 `Ok()` 或 `BadRequest()`），會自動將回應轉換為 ProblemDetails 格式

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack 1.6.3 (17138)
- .NET SDK 8.0.101
- JetBrains Rider 2024.1.4

## 使用方式

1. 基本用法

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

2. 客製 ProblemDetails

    > 以下三種方式

    - ProblemDetailsOptions.CustomizeProblemDetails：適用於增加固定值的情境

        > 在註冊呼叫 `AddProblemDetails` 時，使用 `ProblemDetailsOptions.CustomizeProblemDetails` 來客製化 ProblemDetails，以下透過增加 `producer` 為 `yowko` 來做為範例

        {{<gist yowko f7ae43f2189edc1ead4e39353f7e975b "OptionProblemDetailProgram.cs">}}

        - 實際效果

            ![1optionproblemdetail](https://github.com/user-attachments/assets/f7562423-151b-454f-93b5-b3bf36c26444)

            ![2optionproblemdetail](https://github.com/user-attachments/assets/a91ddf8b-df7e-4cf9-9e63-ef7f4ab3ceaf)

    - IProblemDetailsWriter

        - 實作 IProblemDetailsWriter 介面：Server500ProblemDetailWriter

            > 透過實作 IProblemDetailsWriter 介面來客製化 ProblemDetails，以下透過設定 title 為 `IProblemDetailsWriter:An unexpected error occurred!` 並增加 `producer` 為 `yowko` 來做為範例

            {{<gist yowko f7ae43f2189edc1ead4e39353f7e975b "Server500ProblemDetailWriter.cs">}}

        - 註冊 IProblemDetailsWriter

            > 加入 `builder.Services.AddTransient<IProblemDetailsWriter, Server500ProblemDetailWriter>();` (需要在 `AddRazorPages`, `AddControllers`, `AddControllersWithViews` or `AddMvc` 之前註冊)，其他與基本用法一致

            {{<gist yowko f7ae43f2189edc1ead4e39353f7e975b "IProblemDetailWriterProgram.cs">}}

        - 實際效果

            ![3iproblemdetailswriter](https://github.com/user-attachments/assets/b5e02d6b-5365-44ea-bc48-6aa55cd86bd2)

            ![4iproblemdetailswriter](https://github.com/user-attachments/assets/ccef134d-b467-4072-811a-b0312b248d98)

    - 在 middleware 中使用 IProblemDetailsService.WriteAsync:

        > 因為 middleware 的關係，無法直接攔 exception 或是在 middleware 中使用 try catch，我都紀錄一下做法

        - 官網做法：在處理 request 時在 context 中加入 feature

            {{<gist yowko f7ae43f2189edc1ead4e39353f7e975b "FeatureMiddlewareProblemDetailProgram.cs">}}

            ![5middleware](https://github.com/user-attachments/assets/02ce7a33-6586-49dc-bf49-40fd0c645058)

        - try catch 做法：在 middleware 中使用 try catch 來攔截 exception，需要自行將 status 改為 500，否則為是 200

            {{<gist yowko f7ae43f2189edc1ead4e39353f7e975b "TryMiddlewareProblemDetailProgram.cs">}}

            ![6200middleware](https://github.com/user-attachments/assets/e2945850-6e8b-491e-9793-b85e27ad22ec)

            ![7500middleware](https://github.com/user-attachments/assets/20ee7c1f-994e-4e82-a5bd-10e46186584d)

## 心得

我自己覺得掌握度還是滿低的，沒有深入去理解背後的流程跟設計邏輯，所以在使用上就不清楚到底正確的用法是什麼，只是照著官網的範例來做，或是加加減減自己兜出個會動的範例，初步快速試試，有機會再找找看有沒有更多文件可以參考

## 參考資訊

1. [ASP.NET 8 新增的錯誤處理](/aspnetcore-8-error-handling/)
2. [IETF RFC 7807](https://tools.ietf.org/html/rfc7807)
3. [Microsoft Learn:Handle errors in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594)
4. [Using the ProblemDetails Class in ASP.NET Core Web API](https://code-maze.com/using-the-problemdetails-class-in-asp-net-core-web-api/)
5. [Microsoft Learn:ProblemDetails Class](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.problemdetails?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594)
