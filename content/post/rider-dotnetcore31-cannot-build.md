---
title: "JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄 (更新)"
date: 2019-12-14T15:30:00+08:00
lastmod: 2020-12-11T15:30:31+08:00
draft: false
tags: ["dotnet core","macOS","Tools"]
slug: "rider-dotnetcore31-cannot-build"
---

## JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄 (更新)

前幾天的筆記 [JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄](/rider-dotnetcore3-cannot-build) 提到在升級 .NET Core SDK 3.1.100 我就無法成功透過 JetBrains Rider 來 build project (無論是既有專案或是新建專案)，其間嘗試過重啟 JetBrains Rider 跟 重開機，都一樣無法解決問題，最後調整 JetBrains Rider 的一個設定才看似解決問題，過幾天後同事們接連遭遇相同狀況，在大家協助測試下，大致上解決問題的方式，至於原因還是不明，簡單紀錄一下過程，供日後查閱

過程中我重裝了幾次 .NET SDK，在 .NET Core SDK 3.1.100 與 .NET Core SDK 3.0.100 間重複試驗了幾次，安裝方式可以參考 [使用 Homebrew Cask 安裝舊版本軟體](/homebrew-cask-install-older-version)

## 基本環境說明

1. macOS Catalina 10.15.1
2. .NET Core SDK 3.1.100
3. JetBrains Rider 2019.3 (Build #RD-193.5233.171, built on December 10, 2019)
4. msbuild 16.0.42.19328

## 錯誤訊息

測試了好幾次，錯誤訊息出現過兩種

1. 錯誤一

    - 訊息訊息

        ```txt
        Microsoft.NET.Sdk.FrameworkReferenceResolution.targets(164, 5): [MSB4018] The "GetPackageDirectory" task failed unexpectedly.
        NuGet.Packaging.Core.PackagingException: Unable to find fallback package folder '/usr/local/share/dotnet/sdk/NuGetFallbackFolder'.
            at NuGet.Packaging.FallbackPackagePathResolver..ctor(String userPackageFolder, IEnumerable`1 fallbackPackageFolders)
            at Microsoft.NET.Build.Tasks.NuGetPackageResolver.CreateResolver(IEnumerable`1 packageFolders)
            at Microsoft.NET.Build.Tasks.GetPackageDirectory.ExecuteCore()
            at Microsoft.NET.Build.Tasks.TaskBase.Execute()
            at Microsoft.Build.BackEnd.TaskExecutionHost.Microsoft.Build.BackEnd.ITaskExecutionHost.Execute()
            at Microsoft.Build.BackEnd.TaskBuilder.ExecuteInstantiatedTask(ITaskExecutionHost taskExecutionHost, TaskLoggingContext taskLoggingContext, TaskHost taskHost, ItemBucket bucket, TaskExecutionMode howToExecuteTask)
        ```

    - 錯誤截圖

        ![1error1](https://user-images.githubusercontent.com/3851540/70845881-e4bcf700-1e8e-11ea-9952-e55c77c0d783.png)

2. 錯誤二

    - 訊息內容
  
        ```txt
        Microsoft.NET.TargetFrameworkInference.targets(137, 5): [NETSDK1045] 目前的 .NET SDK 不支援以 .NET Core 3.0 作為目標。請以 .NET Core 2.1 或更低版本作為目標，或是使用支援 .NET Core 3.0 的 .NET SDK 版本。
        ```

    - 錯誤截圖

        ![2error2](https://user-images.githubusercontent.com/3851540/70845882-e5558d80-1e8e-11ea-9239-d142f20d40e7.png)

## 解決方式

1. 將專案 tarket 改為 `netcoreapp3.1`

    > 同事發現將專案改至 `netcoreapp3.1` 就可以通過編譯，但如果再改回 `netcoreapp3.0` 且沒有安裝 .NET Core SDK 3.0 即會出錯

2. 將 JetBrains Rider 徹底關閉

    > 重複試驗時，發現使用 `command + Q` 結束 JetBrains Rider 再重啟後就可以正常編譯，但我在第一次發生問題時沒有用

3. 使用 `dotnet build` 編譯專案

    > 在 JetBrains Rider 的 terminal 中使用指令 `dotnet build {*.sln}` 可能一次通過也可能出錯，但之後便可以正常使用 JetBrains Rider

## 心得

感謝同事們幫忙測試及蒐集實際情境，雖然最後還是不知道問題出在哪裡，以個人不負責任推測是 JetBrains Rider 的 bug，所幸問題不大也不影響開發時程

再次為身為強大團隊的一員感到榮幸，好像沒有什麼問題是團隊解決不了的，大家還會幫忙測試及驗證

## 參考資訊

1. [JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄](/rider-dotnetcore3-cannot-build)
2. [使用 Homebrew Cask 安裝舊版本軟體](/homebrew-cask-install-older-version)
