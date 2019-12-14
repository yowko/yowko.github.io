---
title: "JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄"
date: 2019-12-07T21:30:00+08:00
lastmod: 2019-12-14T15:30:31+08:00
draft: false
tags: ["dotnet core","macOS","Tools"]
slug: "rider-dotnetcore3-cannot-build"
---

## JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄

### <span style="color:red">已有更新版本，請參考 [JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄 (更新)](https://blog.yowko.com/rider-dotnetcore31-cannot-build)<span>

前幾天 .NET Core 3.1 正式 release 了，身為新技術的愛好者，當然是立馬下載安裝試用囉，不過馬上就踩雷了XD 但是其他同事好像沒有遇到類似狀況，我想大概多少還是跟人品有些關聯 QQ

最後問題有解決，但是原因不明，也再無法重現，紀錄一下供日後參考

## 基本環境說明

1. macOS Catalina 10.15.1
2. .NET Core SDK 3.1.100
3. JetBrains Rider 2019.2.3 (Build #RD-192.7317.11, built on October 16, 2019)
4. msbuild 16.0.42.19328

## 問題描述

前幾天升級 macOS Catalina 後，搭配的 .NET Core SDK 3.0.100 都還能正常編譯所有 `TargetFramework` 為 `netcoreapp3.0` 的專案，一升級 .NET Core SDK 3.1.100 後不論是既有專案或是新建專案都無法通過編譯。嘗試過下列方式，問題仍然存在無法解決

1. 重開 Rider
2. 升級 Rider
3. 重開機

## 錯誤訊息

1. 既有專案

    - 錯誤截圖

        ![1existedproject](https://user-images.githubusercontent.com/3851540/70386733-25bc9380-19d7-11ea-8c74-4e5451baea80.png)

    - 訊息內容

        ```txt
        Microsoft.NET.Sdk.FrameworkReferenceResolution.targets(283, 5): [NETSDK1073] The FrameworkReference 'Microsoft.AspNetCore.App' was not recognized
        ```

2. 新建專案

    - 錯誤截圖

        ![2newproject](https://user-images.githubusercontent.com/3851540/70386735-25bc9380-19d7-11ea-89c2-d9f1566cb56e.png)

    - 訊息內容

        ```txt
        Microsoft.Common.CurrentVersion.targets(4570, 5): [MSB3030] Could not copy the file "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/obj/Debug/netcoreapp3.0/TestRedisLuaDictionary" because it was not found.  
        ```

## 問題確認

從上述新建專案的錯誤代碼，看來是 msbuild 的錯誤訊息，所嘗試分別使用 `msbuild` 及 `dotnet cli` 來 build 專案，`dotnet cli` 並未出現錯誤

- msbuild

    ```txt
    Microsoft (R) Build Engine for Mono 16.0.42-preview+g804bde742b 版
    Copyright (C) Microsoft Corporation. 著作權所有，並保留一切權利。

    已經開始建置於 2019/12/6 下午 02:43:22。
    節點 1 (預設目標) 上的專案 "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary.sln"。
    ValidateSolutionConfiguration:
    建置方案組態 "Debug|Any CPU"。
    專案 "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary.sln" (1) 正在節點 1 (預設目標) 上建置 "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/TestRedisLuaDictionary.csproj" (2)。
    /Library/Frameworks/Mono.framework/Versions/5.18.0/lib/mono/msbuild/15.0/bin/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.TargetFrameworkInference.targets(137,5): error NETSDK1045: 目前的 .NET SDK 不支援以 .NET Core 3.0 作為目標。請以 .NET Core 2.1 或更低版本作為目 的 .NET SDK 版本。 [/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/TestRedisLuaDictionary.csproj]
    專案 "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/TestRedisLuaDictionary.csproj" (預設目標) 建置完成 -- 失敗。
    專案 "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary.sln" (預設目標) 建置完成 -- 失敗。

    建置失敗。

    "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary.sln" (預設目標) (1) ->
    "/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/TestRedisLuaDictionary.csproj" (預設目標) (2) ->
    (_CheckForUnsupportedNETCoreVersion 目標) -> 
    /Library/Frameworks/Mono.framework/Versions/5.18.0/lib/mono/msbuild/15.0/bin/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.TargetFrameworkInference.targets(137,5): error NETSDK1045: 目前的 .NET SDK 不支援以 .NET Core 3.0 作為目標。請以 .NET Core 2.1 或更低版本作為.0 的 .NET SDK 版本。 [/Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/TestRedisLuaDictionary.csproj]

        0 個警告
        1 個錯誤

    經過時間 00:00:01.44
    ```

- dotnet cli

    ```txt
    Microsoft (R) Build Engine version 16.4.0+e901037fe for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 321.27 ms for /Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/TestRedisLuaDictionary.csproj.
    TestRedisLuaDictionary -> /Users/yowko.tsai/POCs/TestRedisLuaDictionary/TestRedisLuaDictionary/bin/Debug/netcoreapp3.0/TestRedisLuaDictionary.dll

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:02.74
    ```

## 解決方式

~~由上個動作可以確認是 `msbuild` 造成 JetBrains Rider 無法成功 build 專案的，我的做法是調整了 JetBrains Rider 的 Build Engine~~ 經過測試關鍵似乎是 `dotnet build` 這個指令

- Toolset and Build --> Build Engine --> 取消勾選 `Use ReSharper Build`

    ![3buildengine](https://user-images.githubusercontent.com/3851540/70386736-26552a00-19d7-11ea-957e-88e88e3ffaec.png)

## 心得

開頭有提到我嘗試解決問題的過程中，順利解決問題了就再也無法重現問現問題，指的便是 `調整 Rider 的 Build Engine` 動作，雖然重新勾選 `Use ReSharper Build` 也都可以順利 build project，我個人猜測是不是這個動作重設了 Rider 的設定值，不過既然已經無法重現問題，也無法再往下追查原因，留個紀錄當做備查，但我有想過會不會是 msbuild 版本造成的

## 參考資訊

1. [Fabric MSBuild 1.6.3 MSB3030: Could not copy the file because it was not found](https://github.com/Azure/service-fabric-issues/issues/788)
