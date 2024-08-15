---
title: "在 ASP.NET 中為相同 Interface 註冊多個 Service"
date: 2024-08-15T00:30:00+08:00
lastmod: 2024-08-15T00:30:31+08:00
draft: false
tags: ["csharp","aspdotnet"]
slug: "aspdotnet-one-interface-multiple-services"
---

## 在 ASP.NET 中為相同 Interface 註冊多個 Service

在某些情境下(e.g. 測試目的、不同環境：cloud or on premise 或是依據不同客戶)會需要為相同的 Interface 註冊多個 Service，讓程式可以正確取得所需的 Service

隨著最近團隊將服務升級為 .NET 8，從文件 [Microsoft Learn:Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594) 上看到在 ASP.NET 8 中為相同 Interface 註冊多個 Service 有新的做法，這裡就來簡單紀錄一下

## 基本環境說明

- macOS Sonoma 14.6.1 (Apple M2 Pro)
- OrbStack 1.6.4 (17192)
- .NET SDK 8.0.101
- JetBrains Rider 2024.1.4
- 情境：一個 IEmailService interace，有兩個 Service：SendGridEmailService 與 GmailEmailService

    {{<gist yowko 8b7de8564eb2f865fc847e81bc2b1fe6 "mail.cs">}}

## 使用方式

1. 透過 Func 方式註冊多個 Service

    - 在 Program.cs 註冊多個 Service

        > 可以依實際使用 `AddScoped`、`AddTransient` 或 `AddSingleton` 來註冊多個 Service，以下使用 `AddSingleton` 來示範

        {{<gist yowko 8b7de8564eb2f865fc847e81bc2b1fe6 "FuncProgram1.cs">}}

    - 以 Web Api minimal API 中使用做示範

        {{<gist yowko 8b7de8564eb2f865fc847e81bc2b1fe6 "FuncProgram2.cs">}}

2. 使用 Keyed Services (.NET 8 新增)

    - 在 Program.cs 註冊多個 Service

         > 可以依實際使用 `AddKeyedScoped`、`AddKeyedTransient` 或 `AddKeyedSingleton` 來註冊多個 Service，以下使用 `AddKeyedSingleton` 來示範

        {{<gist yowko 8b7de8564eb2f865fc847e81bc2b1fe6 "KeyedProgram1.cs">}}

    - 以 Web Api minimal API 中使用做示範

        > 使用 `[FromKeyedServices]` 指定需要的 Service

        {{<gist yowko 8b7de8564eb2f865fc847e81bc2b1fe6 "KeyedProgram2.cs">}}

## 心得

雖然 .NET 8 的 Keyed Services 看起來更簡潔，意圖也更明確，但在實際使用上，多一個 Attribute `[FromKeyedServices]` 來指定需要的 Service 增加了些複雜度，所以實際應用上我倒不是完全偏向 Keyed Services，我個人覺得差不多，看實際情境來決定使用哪種方式

## 參考資訊

1. [Microsoft Learn:Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594)
2. [.NET 8 Dependency Injection Changes: Keyed Services](https://weblogs.asp.net/ricardoperes/net-8-dependency-injection-changes-keyed-services?WT.mc_id=DOP-MVP-5002594)
3. [Keyed Services in .NET 8 Dependency Injection](https://medium.com/@nirajranasinghe/keyed-services-in-net-8-dependency-injection-80a17fbe4b20)
4. [Registering Multiple Services with a Single Interface in .NET Core](https://touseefkhan4pk.medium.com/registering-multiple-services-with-a-single-interface-in-net-core-e6e4a1a0ec04)
