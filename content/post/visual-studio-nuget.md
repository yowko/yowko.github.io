---
title: "Visual Studio 中無法使用 NuGet 指令？！"
date: 2017-07-28T00:19:00+08:00
lastmod: 2018-09-24T00:26:00+08:00
draft: false
tags: ["NuGet","Visual Studio"]
slug: "visual-studio-nuget"
aliases:
    - /2017/07/visual-studio-nuget.html
---
# Visual Studio 中無法使用 NuGet 指令？！
在測試 NuGet 相關參數時，竟然發現在 Visual Studio 中無法使用 NuGet 指令，原本以為 NuGet 功能故障但令我更驚訝的是可以正常使用 NuGet 來管理套件，只是無法使用 NuGet 相關的指令

經過一番資料查閱後，才發現原來 NuGet 還細分為不同的 component：NuGet 管理套件功能屬於 Visual Studio Extension 的一部份，而 NuGet 指令則屬於 NuGet command-line tool，所以才會出現部份功能異常的情況

## About NuGet

NuGet 組件的基本介紹

1.  [NuGet client tools](https://github.com/nuget/nuget.client)
    *   NuGet command-line tool 3.0 and higher
    *   Visual Studio 2015 Extension
    *   PowerShell CmdLets

2.  [NuGet V2](https://github.com/NuGet/NuGet2)
    *   NuGet command-line tool 2.9
    *   Visual Studio Extension (Previous versions e.g. Visual studio 2013)
    *   PowerShell CmdLets
    *   NuGet.Core

## 錯誤訊息

*  訊息內容
    
    ```
    nuget : The term 'nuget' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path as included, verify that the path is correct and try again.
        At line:1 char:1
        + nuget help
        + ~~~~~
        + CategoryInfo          : ObjectNotFound: (nuget:String) [],    ommandNotFoundException
        + FullyQualifiedErrorId : CommandNotFoundException
    ```

*   錯誤訊息截圖

    ![1error](https://user-images.githubusercontent.com/3851540/28726331-7079311c-73f3-11e7-964b-8e6d58aa0136.png)

## 解決方式

1.  將 `NuGet.exe` 的資料夾路徑加至環境變數 `PATH` 中

    > 適用於各個版本的 Visual Studio

2.  安裝 `NuGet.CommandLine`

    > `Install-Package NuGet.CommandLine`

    *   需指定安裝 project
    *   一個 solution 中只需有個 project 安裝即可
    *   適用於 Visual Studio 2015 及之前版本，Visual Studio 2017 並不適用


*   正常執行

    ![2noerror](https://user-images.githubusercontent.com/3851540/28726332-70a7e8b8-73f3-11e7-9506-3176346a2bb0.png)


## 心得

在公司電腦並沒有遇到相同問題，但每台個人電腦卻都有一樣狀況，花了不少時間才發現因為公司電腦有安裝 Jenkins 所以早早就把 NuGet.exe 加入環境變數中，造成第一時間沒注意這個差異

另外最近遇到幾個問題，Visual Studio 2017 的解決方式都與之前版本不同(ex. 外部工具的用法、MSTestV2 的指令行為...)，雖然 Visual Studio 2017 在效能表現很優異，但潛在的小問題還是有，而且一遇到都不太好查，希望可以快點獲得改善

# 參考資訊

1.  [nuget](https://github.com/nuget/home)
2.  [NuGet client tools](https://github.com/nuget/nuget.client)
3.  [NuGet V2](https://github.com/NuGet/NuGet2)
4.  ['nuget' is not recognized but other nuget commands working](https://stackoverflow.com/questions/13056329/nuget-is-not-recognized-but-other-nuget-commands-working)
5.  [Available NuGet Distribution Versions](https://dist.nuget.org/index.html)
