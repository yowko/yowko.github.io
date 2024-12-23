---
title: "NuGet Central Package Management 集中套件管理，簡化套件相依管理"
date: 2024-12-23T00:30:00+08:00
lastmod: 2024-12-23T00:30:31+08:00
draft: false
tags: ["dotnet","csharp","nuget"]
slug: "nuget-central-package-management"
---

## NuGet Central Package Management 集中套件管理，簡化套件相依管理

隨著專案日漸擴大，package 相依性也越來越複雜，尤其是在多個專案中使用相同的 package，當 package 更新時，需要逐一更新每個專案的 package 版本，這樣的管理方式不僅耗時，也容易出錯。過去我所在的團隊是透過建立一個獨立專案，將所有的 package 版本集中管理，不過這樣的方式還是有些缺點：

1. 每個專案還是需要手動更新這個獨立的 package 專案版本
2. 特定專案可能需要特定版本的 package，就會遇到版本衝突問題
3. 多個專案互相依賴 (各自有自己的 package 專案與版本)，可能會造成 package 版本衝突或是多了額外的 package

為了解決 package 相依管理的問題，Microsoft 推出了 NuGet Central Package Management 功能，讓開發者可以在專案中集中管理套件版本，這樣就可以在多個專案中共用相同的套件版本，不僅可以簡化套件相依管理，也可以提高開發效率。

NuGet Central Package Management 是隨著 .NET 6 搭配 NuGet 6.2 推出的功能，從以下版本開始提供支援

1. Visual Studio 2022 17.2
2. JetBrains Rider 2022.3
3. .NET SDK 6.0.300
4. nuget.exe 6.2.0

使用上有其規則：

1. 一個專案只會套用一個 `Directory.Packages.props` 檔案
2. 如果 repository 中有多個 `Directory.Packages.props` 檔案，最靠近專案的 `Directory.Packages.props` 檔案會被採用

## 基本環境說明

1. macOS Sequoia 15.1.1 (Apple M2 Pro)
2. .NET SDK 9.0.1
3. NuGet 6.12.0.127
4. NuGet Packages

    - StackExchange.Redis 2.8.24
    - RabbitMQ.Client 7.0.0
    - MongoDB.Driver 3.1.0

## 設定方式

1. 在 solution 方案根目錄中建立 `Directory.Packages.props` 檔案並設定 MSBuild 屬性 `ManagePackageVersionsCentrally` 為 `true`

    {{<gist yowko c0652f649544baeaabcbd3d5c0720b09 "Directory.Packages.props-default">}}

2. 在 `Directory.Packages.props` 加入 `<PackageVersion />`

    > `<PackageVersion Include="{{套件名稱}}" Version="{{套件版本}}" />` 相較舊版的 `<PackageReference Include="{{套件名稱}}" Version="{{套件版本}}"/>`，差別是在 property 名稱改為 `PackageVersion`

    {{<gist yowko c0652f649544baeaabcbd3d5c0720b09 "Directory.Packages.props-redis">}}

3. 在一般專案中加入沒有版本資訊的 `<PackageReference />`

    > `<PackageReference Include="{{套件名稱}}"/>` 相較舊版的 `<PackageReference Include="{{套件名稱}}" Version="{{套件版本}}"/>`，新版的 NuGet 會自動套用 `Directory.Packages.props` 中的版本資訊

    {{<gist yowko c0652f649544baeaabcbd3d5c0720b09 "ProjectA.csproj">}}

## 其他用法

- 在專案中覆寫 `Directory.Packages.props` 中的版本

    > 在專案中加入 `<PackageReference />` 並設定 `VersionOverride` 屬性為所需版本，這樣專案中的套件版本就會以專案中的設定版本為主

    {{<gist yowko c0652f649544baeaabcbd3d5c0720b09 "ProjectA.csproj-override">}}

- 全域引用套件 (Global Package References)

    > 在 `Directory.Packages.props` 中加入 `<GlobalPackageReference Include="{{套件名稱}}" Version="{{套件版本}}" />` 即會為 solution 中的所有專案加上指定的套件版本

    - 屬性：`IncludeAssets="Runtime;Build;Native;contentFiles;Analyzers"`：確保 package 僅用作開發 dependency，並防止任何編譯時程式集引用

    - 屬性：`PrivateAssets="All"`：避免下游 dependency 使用到全域引用套件

    {{<gist yowko c0652f649544baeaabcbd3d5c0720b09 "Directory.Packages.props-global">}}

    - 條件：
        - Visual Studio 2022 17.4 或以上版本 (Rider 沒找到資料)
        - .NET SDK 7.0.100.preview7 或以上版本
        - NuGet 6.4 或以上版本

## 心得

雖然一樣在 solution 方案根目錄中建立 `Directory.Packages.props`，但我在 Rider 的 `File System` explorer 中加入 `Directory.Packages.props` 檔案後，Rider 並沒有自動套用 `Directory.Packages.props` 中的套件版本，需要手動 reload all projects 才會生效，或是使用 `Solution` explorer 來加入 `Directory.Packages.props` 就沒有這個問題，應該是 Rider 沒有偵測到 `File System` explorer 中的檔案變更造成的。

初步用起來，Central Package Management 功能確實可以簡化套件相依管理，尤其是在多個專案中使用相同的套件版本，不用再逐一更新每個專案的套件版本，只需要在 `Directory.Packages.props` 中設定套件版本，就可以自動套用到所有專案中，目前沒有遇到什麼問題，實際使用後有發現問題再補充。

完整歷程可以參考 [GitHub:yowko/NuGet-Central-Package-Management-Demo](https://github.com/yowko/NuGet-Central-Package-Management-Demo)

## 參考資訊

1. [Microsoft:Introducing Central Package Management](https://devblogs.microsoft.com/nuget/introducing-central-package-management/?WT.mc_id=DOP-MVP-5002594)
2. [Microsoft Learn:Central Package Management](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management?WT.mc_id=DOP-MVP-5002594)
3. [NuGet Central Package Management Comes To JetBrains Rider](https://blog.jetbrains.com/dotnet/2022/11/07/nuget-central-package-management-comes-to-jetbrains-rider/)
4. [Central Package Management in .NET — Simplify NuGet Dependencies](https://medium.com/@MilanJovanovicTech/central-package-management-in-net-simplify-nuget-dependencies-1f6c744f79d7)
5. [Centralize your packages in .NET with NuGet](https://steven-giesel.com/blogPost/988c047a-914a-446d-bd28-4840643dcfb9/centralize-your-packages-in-net-with-nuget?_bhlid=465671ab6dec7750e5d98b3df2ff1fc6df575d58)
6. [GitHub:yowko/NuGet-Central-Package-Management-Demo](https://github.com/yowko/NuGet-Central-Package-Management-Demo)
