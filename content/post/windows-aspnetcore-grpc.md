---
title: "在 Windows 上的 ASP.NET Core 中呼叫 gRPC"
date: 2020-02-24T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["Windows","dotnet core","gRPC"]
slug: "windows-aspnetcore-grpc"
---

## 在 Windows 上的 ASP.NET Core 中呼叫 gRPC

現在的工作主力都在 mac 上，但最近在測試功能時覺得與過去認知不同，特別用 Windows 測試一下，證實在 Windows 平台上功能與印象相同，為了日後比較方便，筆記一下

## 基本環境說明

1. Windows 10 Pro 1903 (OS Build 18362.657)
2. .NET Core SDK 3.1.102
3. Visual Studio 2019 (16.4.5)
4. 專案結構與 NuGet 套件

    - GpcService (gRPC Service 預設專案範本)
      - Grpc.AspNetCore 2.27.0
    - GrpcMessages (.NET Core class library - 用來存放 *.proto)
      - Google.Protobug 3.11.4
      - Grpc.Net.Client 2.270.0
      - Grpc.Tools 2.27.0
    - GrpcClient (ASP.NET Core MVC 預設專案範本)
      - Grpc.AspNetCore 2.27.0

## 設定方式

這就是我在 macOS 與 Windows 兩個平台開發上覺得疑惑的點，Windows 上的設定非常簡單：

1. GpcService 設定開發憑證

    > 這個不得不稱讚 Microsoft 在 Visual Studio 上的整合非常到位，透過 Visual Studio 2019 (其他 Visual Studio 版本，我沒有測試) 啟動 `GpcService` 就會提示加入與信任開發憑證

    ![1cert1](https://user-images.githubusercontent.com/3851540/75628253-1cc41280-5c12-11ea-8bbb-b196ee817b04.png)

    ![2cert2](https://user-images.githubusercontent.com/3851540/75628255-1e8dd600-5c12-11ea-9a52-fcdec4305fba.png)

2. GrpcClient 設定 GpcService 的 endpoint

    > 在 `Startup.cs` 的 `ConfigureServices` 方法中加入以下程式碼即可，與 [在 macOS 上的 ASP.NET Core 中呼叫 gRPC](/macos-aspnetcore-grpc) 比較就知道設定差異了

    ```cs
    services.AddGrpcClient<GrpcMessges.Greeter.GreeterClient>(o =>
        {
            o.Address = new Uri("https://localhost:5001");
        });
    ```

## 心得

我個人覺得 Windows 在整體的使用上比較合理，不需要因為平台不同而加入必要的程式碼，雖然我知道這跟平台基礎架構有關，但還是覺得有些困擾，畢竟開發環境跟實際的 production 環境不是走相同 flow，多少有些擔心

## 參考資訊

1. [搭配 ASP.NET Core 的 gRPC 服務](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/aspnetcore?view=aspnetcore-3.1&tabs=visual-studio&WT.mc_id=DOP-MVP-5002594)
2. [.NET Core 上的 gRPC 簡介](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594)
3. [教學課程：在 ASP.NET Core 中建立 gRPC 用戶端和伺服器](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-3.1&tabs=visual-studio&WT.mc_id=DOP-MVP-5002594)
4. [在 macOS 上的 ASP.NET Core 中呼叫 gRPC](/macos-aspnetcore-grpc)
