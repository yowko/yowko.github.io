---
title: "NuGet 4.0 的特色"
date: 2017-03-15T02:43:34+08:00
lastmod: 2020-09-01T00:42:34+08:00
draft: false
tags: ["NuGet"]
slug: "nuget-4"
aliases:
    - /2017/03/nuget-4.html
---
# NuGet 4.0 的特色
偶然間看到發哥在 facebook 揭露 NuGet 4.0 RTM 的消息，身為 .NET 工程師一定要來瞭解其中的特色呀

## 重要改變
1. NuGet Package Manager 成為 Visual Studio 的一部份
    
    >從 Visual Studio 2017 的 NuGet 4.0 開始， NuGet Package Manager 將與 Visual Studio 綁在一起，不再支援從 VS extensions 下載，NuGet 更新將與 Visual Studio 更新一起
2. 不同的 NuGet.config 預設位置
    
    >因為安全性問題(原本放在 `%ProgramData%\NuGet\Config\` 中不需要管理權限即可修改)，從 Visual Studio 2017 開始， NuGet.config 預設位置 將調整至 `%ProgramFiles(x86)%\NuGet\Config\`。原存在 `%ProgramData%\NuGet\Config\` 的相關設定可以手動搬至新位置

## 新功能
1. PackageReference
    
    >PackageReference 將成為 NuGet 管理套件相依性的新標準，NuGet 4.0 將完整支援 .NET Core 專案，這讓我們可以使用條件式編譯為不同的 framework,configuration,platform,其他重要變數來使用不同的套件
    
    ```xml
    <ItemGroup>
        ...
        <PackageReference Include="Contoso.Utility.UsefulStuff" Version="3.6.0"/>
        ...
    </ItemGroup>
    ```
    - 關於套件相依性管理補充如下
        - NuGet 2.x 使用 packages.config
        - NuGet 3.x 使用 project.json
2. 內建至 MSBuild 中
    - restore
        
        > CI Server 不需再額外下載 nuget.exe，可以直接使用 msbuild 來 restore package
            
        ```
        nuget restor = msbuild /t:restore
        ```
    - pack
        
        > 不用再使用 nuget 指令來產生 nuget packaget，可以直接使用 msbuild 來產生 nuget package
            
        ```
        nuget pack = msbuild /t:pack 
        ```

3. 背景管理套件
    
    `PackageReference` 內容如果有異動就會背景同步處理 package，不需要像以前手動執行還原或透過 build 來還原
4. 加入至 .NET CLI 指令中
    - dotnet nuget locals
        
        >操作 local 的 nuget cache 
    - dotnet nuget push
        
        >可以將 package 發行至 nuget server 
    - dotnet nuget delete 
        
        >可以從 nuget server 刪除 package
    - dotnet restore
        
         >即 `msbuild /t:restore`   
    - dotnet pack
        
         >即 `msbuild /t:pack` 
5. 執行效能改善
    - .NET Core 的 tool restore 從原本的一個 project restore 一次優化一個 solution restore 一次
    - nuget update 
        - 使用 project.json 的專案，官方數據約是之前的 6 倍
        - 使用 packages.confg 的專案，官方數據約快了 20% 
    - nuget restore
        - 使用 project.json 的專案，官方數據 restore 時間從 3800ms 減少至 400ms
        - 使用 packages.confg 的專案，官方未提供數據

# 參考資料
1. [Announcing NuGet 4.0 RTM](http://blog.nuget.org/20170308/Announcing-NuGet-4.0-RTM.html)
2. [Dependency Resolution](https://docs.microsoft.com/en-us/nuget/consume-packages/dependency-resolution?WT.mc_id=DOP-MVP-5002594)
3. [Creating a Package](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package?WT.mc_id=DOP-MVP-5002594)
4. [4.0 RTM Release Notes](https://docs.microsoft.com/en-us/nuget/release-notes/nuget-4.0-rtm?WT.mc_id=DOP-MVP-5002594)