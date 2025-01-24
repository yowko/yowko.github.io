---
title: "ASP.NET minimal API 解析 text/json"
date: 2025-01-24T00:30:00+08:00
lastmod: 2025-01-24T00:30:31+08:00
draft: false
tags: ["csharp","aspdotnet"]
slug: "aspnetcore-minimal-api-text-json"
---

## ASP.NET minimal API 解析 text/json

這個需求一樣來自之前筆記 [ASP.NET minimal API 將多個 route pattern 套用相同處理邏輯](/aspnetcore-minimal-api-multiple-route-patterns) 中所提到的舊系統，不過這次不是打錯字XD 舊系統使用 web api controller 來接收 post 傳遞的 model，一開始沒有特別指定 content-type，所以有的呼叫端使用 `application/json` 有的則是使用 `text/json`，測試皆可正常運作，這次改用 minimal api 卻在 `Content-Type: text/json` 的 request 中收到 `415 Unsupported Media Type` 錯誤

雖然沒有明確的文件表示是 minimal api 行為修改造成的，但實際測試結果確實存在使用 controller 可以正確完成 `application/json`、`text/json` 的 model binding，但是使用 minimal api 卻無法正確執行 `text/json` 的 modle binding，所以這次就來紀錄一下該如何處理。

## 基本環境說明

- macOS Sequoia 15.2 (Apple M2 Pro)
- .NET SDK 9.0.100
- JetBrains Rider 2024.3.3
- 原始程式碼

    - TestController.cs

        {{<gist yowko 952977f706e9c9a5b252da35a5423937 "Controller.cs">}}

    - Program.cs

        {{<gist yowko 952977f706e9c9a5b252da35a5423937 "Program.cs">}}

    - TextModel.cs

        {{<gist yowko 952977f706e9c9a5b252da35a5423937 "TextModel.cs">}}

    - http request via curl

        {{<gist yowko 952977f706e9c9a5b252da35a5423937 "request.sh">}}

## 設定方式：手動判斷 Content-Type 並轉換

{{<gist yowko 952977f706e9c9a5b252da35a5423937 "Program-manual.cs">}}

## 心得

1. 嘗試了使用 [Accept Content-Type:text/json request in Minimal API](https://stackoverflow.com/a/78083376) 提到的在 `.MapPost` 方法後面加上 `.Accepts(typeof(TextModel), "text/json")` 還是收到 415 Unsupported Media Type 的錯誤

    {{<gist yowko 952977f706e9c9a5b252da35a5423937 "Program-accept.cs">}}

2. 在手動處理 content-type 如果有額外指定 charset 時，在 `Request.Headers["Content-Type"]` 取得的值會是 `text/json; charset=utf-8`，需要特別處理，取得 `text/json` 的部分，這樣才能正確判斷 content-type；不過使用 `application/json; charset=utf-8` 預設就可以正常完成 model binding；另外 controller 版本也可以正常完成 model binding
3. 這次的解法是使用 `Request.Headers["Content-Type"]` 來判斷 Content-Type，這樣的方式雖然可以解決問題，但是感覺不太乾淨，希望未來能有更好的解法

完整範例程式碼可以參考 [GitHub - yowko/multiple-route-text-json](https://github.com/yowko/multiple-route-text-json)

## 參考資訊

1. [ASP.NET minimal API 將多個 route pattern 套用相同處理邏輯](/aspnetcore-minimal-api-multiple-route-patterns)
2. [Accept Content-Type:text/json request in Minimal API](https://stackoverflow.com/a/78083376)
3. [GitHub Issues - Improve Minimal APIs support for request media types & body schema/shape](https://github.com/dotnet/aspnetcore/issues/35082)
4. [GitHub - yowko/multiple-route-text-json](https://github.com/yowko/multiple-route-text-json)
