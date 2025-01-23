---
title: "ASP.NET minimal API 將多個 route pattern 套用相同處理邏輯"
date: 2025-01-22T00:30:00+08:00
lastmod: 2025-01-22T00:30:31+08:00
draft: false
tags: ["csharp","aspdotnet"]
slug: "aspnetcore-minimal-api-multiple-route-patterns"
---

## ASP.NET minimal API 將多個 route pattern 套用相同處理邏輯

這個需求來自一個內部共用服務的 API 上，之前同事在建立處理邏輯時，不慎將 controller name 打錯，預計使用 `Text` 但打成 `Test`，導致其他服務在呼叫時，都需要呼叫這個 `Test` 的 endpoint，這樣的命名對於呼叫端一定會存疑：這個 endpoint 是測試用的嗎？應該不是正式的吧？！ 雖然有心想要破釜沈丹將 type 修正，不過強制要求已經完成介接的系統一併修改不太實際，而我也不想讓這個錯誤一直存在，所以我便在既有的 `Test` endpoint 上，新增一個 `Text` 的 endpoint，讓呼叫端可以同時呼叫這兩個 endpoint，並且在過渡期間，逐步將呼叫端的程式碼修改過來。

{{<gist yowko b25345c481d26cfb893fcacb8a0af538 "TestController.cs">}}

近期因為內部服務對外的警示 IM 改用 Telegram 所以擔任中介角色的 API 也趁這次機會一併使用 ASP.NET minimal API 改寫，但是在改寫的過程中，卻在如何將多個 route pattern 套用相同的處理邏輯上遇到了障礙，所以今天就來紀錄一下該如何處理

## 基本環境說明

- macOS Sequoia 15.2 (Apple M2 Pro)
- .NET SDK 9.0.100
- JetBrains Rider 2024.3.3

## 將多個 route pattern 套用相同處理邏輯

1. 將處理邏輯使用 method 來包裝

    {{<gist yowko b25345c481d26cfb893fcacb8a0af538 "Program-method.cs">}}

2. 建立 handler dictionary：儲存處理邏輯

    {{<gist yowko b25345c481d26cfb893fcacb8a0af538 "Program-handler.cs">}}

## 心得

相較 controller 可以直接使用 attribute 來設定 route pattern 絕對還是方便許多，但是 minimal api 透過 method 或是 handler dictionary 來儲存處理邏輯，雖然在程式碼上看來步驟與內容比較多，但是也能夠達到相同的效果。我自己是覺得對於 `意圖` 的呈現 attribute 還是比較直覺

## 參考資訊

1. [Microsoft Learn:Route Handlers in Minimal API apps](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-9.0&WT.mc_id=DOP-MVP-5002594)
