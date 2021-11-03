---
title: "列出當下環境可以使用的 .NET Core 專案範本"
date: 2018-11-18T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["dotnet core"]
slug: "dotnet-template-list"
---
## 列出當下環境可以使用的 .NET Core 專案範本

.NET Core 問世以來，跨平台的特性帶來了許多的改變，其中一個就是開發模式，過去 .Net 開發人員只能透過 Visual Studio 這類的 IDE 工具進行開發，但跨平台之後使用 IDE 不一定是所有開發人員的首選，畢竟單論執行效率上 IDE 難免還是有些延遲，加上 Linux 環境也不一定有 GUI 介面可以使用，綜合以上原因，身為一個熟悉 Visual Studio 的開發人員也得試著上手相關指令

過去想要得知道有什麼專案範本，最直接快速的方法就是打開 Visual Studio ，立馬一目了然，但如果沒有辦法使用 Visual Studio 時該如何是好呢？！因此我的第一個 .NET Core 專案就卡住了XD 所以今天就來紀錄一下如何列出所有當下環境中可以使用的 .NET Core 專案範本

## 環境說明

* .NET Core SDK 版本 ：`2.1.403`

    > 使用 `dotnet --version` 可以取得 `.NET Core SDK 版本`

    ![1version](https://user-images.githubusercontent.com/3851540/48675246-9b1db200-eb91-11e8-9bcb-aeea1d737bee.png)

* OS 環境：Windows 10 1803 (OS Build 17134.254)

## 列出現有可使用專案範本

1. SDK 預設安裝範本

    > 以 `2.1.403` 為例

    * `dotnet new -l` 或是 `dotnet new --list` ，兩者行為相同

        > * 名稱顯示差異： `範本描述` 是 microsoft Docs 上的顯示名稱，在 console 會顯示 `Templates`; `範本名稱` 在 console 上為 `Short Name`
        > * 支援語言標示：有使用 `[]` 標記的為該範本預設使用的語言
        > * 文件與實際行為不同：僅會列出指定資料夾可用的專案範本，但實際上仍會列所有專案範本

        ![3listerror](https://user-images.githubusercontent.com/3851540/48675249-9b1db200-eb91-11e8-9062-d156b7fd7940.png)

        範本描述|範本名稱|支援語言|Tags
        ---|---|---|---
        Console Application|console|`[C#]`, F#, VB|Common/Console
        Class library|classlib| `[C#]`, F#, VB| Common/Library
        Unit Test Project|mstest| `[C#]`, F#, VB| Test/MSTest
        NUnit 3 Test Project|nunit| `[C#]`, F#, VB| Test/NUnit
        NUnit 3 Test Item|nunit-test| `[C#]`, F#, VB| Test/NUnit
        xUnit Test Project|xunit| `[C#]`, F#, VB| Test/xUnit
        Razor Page| page| `[C#]`| Web/ASP.NET
        MVC ViewImports| viewimports| `[C#]`| Web/ASP.NET
        MVC ViewStart| viewstart| `[C#]` | Web/ASP.NET
        ASP.NET Core Empty|web| `[C#]`, F#| Web/Empty
        ASP.NET Core Web App (Model-View-Controller)| mvc| `[C#]`, F#| Web/MVC
        ASP.NET Core Web App| razor| `[C#]`| Web/MVC/Razor Pages
        ASP.NET Core with Angular| angular| `[C#]`| Web/MVC/SPA
        ASP.NET Core with React.js| react| `[C#]`| Web/MVC/SPA
        ASP.NET Core with React.js and Redux| reactredux| `[C#]`| Web/MVC/SPA
        Razor Class Library| razorclasslib | `[C#]`| Web/Razor/Library/Razor Class Library
        ASP.NET Core Web API| webapi| `[C#]`, F# | Web/WebAPI
        global.json file| globaljson|| Config
        NuGet Config| nugetconfig| |Config
        Web Config |webconfig| |Config
        Solution File| sln| |Solution

        ![2newlist](https://user-images.githubusercontent.com/3851540/48675247-9b1db200-eb91-11e8-9303-d6ffaeebc97f.png)

2. 過濾指定語言

    * `dotnet new -l -lang {C#|F#|VB}` 或是 `dotnet new -l --language {C#|F#|VB}` ，兩者行為相同
    * 實測指令在 Windows 環境下 (cmd 與 powershell) 都不需使用 `""` 將語言包住，可以正確解析 `#`
    * 指定空白串，會直接列出所有專案範本而不是沒有語言類型 (solution 、config...) 這類範本

    ![4lang](https://user-images.githubusercontent.com/3851540/48675250-9bb64880-eb91-11e8-86ae-930c6b000445.png)

3. 過濾指定類型

    * `dotnet new -l --type {project|item|other}`

        * `dotnet new -l --type project`

            範本描述|範本名稱|支援語言|Tags
            ---|---|---|---
            Console Application|console|`[C#]`, F#, VB|Common/Console
            Class library|classlib| `[C#]`, F#, VB| Common/Library
            Unit Test Project|mstest| `[C#]`, F#, VB| Test/MSTest
            NUnit 3 Test Project|nunit| `[C#]`, F#, VB| Test/NUnit
            xUnit Test Project|xunit| `[C#]`, F#, VB| Test/xUnit
            ASP.NET Core Empty|web| `[C#]`, F#| Web/Empty
            ASP.NET Core Web App (Model-View-Controller)| mvc| `[C#]`, F#| Web/MVC
            ASP.NET Core Web App| razor| `[C#]`| Web/MVC/Razor Pages
            ASP.NET Core with Angular| angular| `[C#]`| Web/MVC/SPA
            ASP.NET Core with React.js| react| `[C#]`| Web/MVC/SPA
            ASP.NET Core with React.js and Redux| reactredux| `[C#]`| Web/MVC/SPA
            Razor Class Library| razorclasslib | `[C#]`| Web/Razor/Library/Razor Class ASP.NET Core Web API| webapi| `[C#]`, F# | Web/WebAPI
            Solution File| sln| |Solution

            ![5project](https://user-images.githubusercontent.com/3851540/48675240-99ec8500-eb91-11e8-940d-2bf32a5a1cc8.png)

        * `dotnet new -l --type item`

            範本描述|範本名稱|支援語言|Tags
            ---|---|---|---
            NUnit 3 Test Item|nunit-test| `[C#]`, F#, VB| Test/NUnit
            Razor Page| page| `[C#]`| Web/ASP.NET
            MVC ViewImports| viewimports| `[C#]`| Web/ASP.NET
            MVC ViewStart| viewstart| `[C#]` | Web/ASP.NET
            global.json file| globaljson|| Config
            NuGet Config| nugetconfig| |Config
            Web Config |webconfig| |Config
            Solution File| sln| |Solution

            ![6item](https://user-images.githubusercontent.com/3851540/48675241-9a851b80-eb91-11e8-8f5b-cbc481429361.png)

        * `dotnet new -l --type other`

            範本描述|範本名稱|支援語言|Tags
            ---|---|---|---
            Solution File| sln| |Solution

            ![7other](https://user-images.githubusercontent.com/3851540/48675242-9a851b80-eb91-11e8-95f9-575cc146ea6e.png)

4. 混合過濾語言及類型

    * `dotnet new -l --type {project|item|other} -lang {c#|vb|f#}`

        ![8projectlang](https://user-images.githubusercontent.com/3851540/48675243-9a851b80-eb91-11e8-84da-002517cf9559.png)

5. 列出某個 template 名稱下的所有 template

    * `dotnet new {template name} -l`

        ![9templatelist](https://user-images.githubusercontent.com/3851540/48675244-9a851b80-eb91-11e8-830d-f6dd460dbe2a.png)

6. 未符合 template 名稱時自動模糊比對

    * `dotnet new {not a template name} -l`

        ![10partialmatch](https://user-images.githubusercontent.com/3851540/48675245-9b1db200-eb91-11e8-8d32-aae3bce48c9f.png)

## 心得

.NET Core CLI 指令的即時回饋比起 IDE 確實快了許多，加上有了跨平台的優勢，相信 .NET Core CLI 必然會更加普及也會更佳完善

目前使用上並沒有遇到什麼大問題，但難免 wording 使用上還有改善空間，另外就是實際行為與文件不相符的問題，但這些小狀況都瑕不掩瑜，整體上還是相當值得推薦使用

## 參考資訊

1. [dotnet new](https://docs.microsoft.com/zh-tw/dotnet/core/tools/dotnet-new?WT.mc_id=DOP-MVP-5002594)
