---
title: "NuGet 設定 Insecure HTTP source"
date: 2023-11-23T00:30:00+08:00
lastmod: 2023-11-23T00:30:31+08:00
draft: false
tags: ["dotnet8","dotnet","NuGet"]
slug: "nuget-insecure"
---

## NuGet 設定 Insecure HTTP source

之前筆記 [NuGet restore error NU1803](/nuget-restore-nu1803) 紀錄了在某次 build fail 後意外發現 Microsoft NuGet team 的 [HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594) 計劃，眼看著計劃中的時程慢慢接近，正在進行相應計劃：包含忽略 NU1803 error 與內部 dns 以 apply https，查資料時看到 Microsoft NuGet team 的新計劃：[HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594) 從 NuGet 6.8 開始允許啟用 Insecure NuGet source，所以今天就來紀錄一下如何設定 NuGet Insecure HTTP source

## 基本環境說明

1. macOS Sonoma 14.1.1 (Apple M2 Pro)
2. Windows 10 Pro 22H2
3. .NET SDK 8.0.100
4. NuGet 6.8.0.122

    > `dotnet nuget --version`

5. JetBrains Rider 2023.2.3
6. Microsoft Visual Studio Enterprise 2022 (64-bit) - Current Version 17.8.1
7. 透過 nexus 來模擬 NuGet HTTP source

    > `docker run -d -p 8081:8081 --name nexus sonatype/nexus3`

## 設定方式：為 NuGet source 加上 `allowInsecureConnections="true"`

- NuGet.Config 有三個層級：

    - Solution：會套用至所有子資料夾中的專案，但非專案層級設定無法只生效於特定專案
        > 實測下，NuGet.Config 與 .sln 檔案同資料即可套用，不需將檔案設定為 Solution Item
    - User：適用於特定使用者的所有操作，會由 Solution 設定所覆寫
    - Computer：適用於電腦上所有使用者的任何操作，會由 User 設定或 Solution 設定 所覆寫

1. 手動修改 NuGet.Config 為 NuGet source 加上 `allowInsecureConnections="true"`

    - 原始 設定

        ```xml
        <add key="localInsecure" value="http://localhost:8081/repository/nuget-hosted/"/>
        ```

    - 修改後 設定

        ```xml
        <add key="localInsecure" value="http://localhost:8081/repository/nuget-hosted/" allowInsecureConnections="true"/>
        ```

2. 實際範例

    {{< gist yowko 6dad20810099b49df0f546ea098656cf >}}

## 心得

1. 相同的 .NET SDK 8.0.100 與 NuGet 6.8.0.122 甚至同個專案，但在 macOS 上 `allowInsecureConnections="true"` 的設定並不會生效：使用 dotnet cli 與 JetBrains Rider 都會出現 `Warning NU1803`

    - dotnet cli

        ![1dotnetcli](https://github.com/yowko/picsbed/assets/3851540/f6f7437e-31af-474c-af6a-d41d15416baf)

    - JetBrains Rider

        ![2rider](https://github.com/yowko/picsbed/assets/3851540/57bf54b1-b9a3-4c10-b826-2b2b092c43be)

2. 除此之外，JetBrains Rider build 有很強的 cache 效果，同個專案 build 第二次就不會出現 `Warning NU1803`，但使用 dotnet cli 則不會有這個問題，我自己是重新增減 nuget library 來避免 cache 的影響。JetBrains Rider 啟動時有提示暫時尚未完全支援 .NET 8，不知道是不是這個原因

3. JetBrains Rider 跟 Visual Studio 2022 都有提供 NuGet.Config 的編輯介面，但兩者在介面上都沒有 `allowInsecureConnections="true"` 的設定，都需要用文字編輯器修改 NuGet.Config

## 參考資訊

1. [NuGet Warning NU1803](https://docs.microsoft.com/en-us/nuget/reference/errors-and-warnings/nu1803?WT.mc_id=DOP-MVP-5002594)
2. [HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594)
3. [NuGet restore error NU1803](/nuget-restore-nu1803)
4. [HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594)
5. [NuGet 6.8](https://docs.microsoft.com/nuget/release-notes/nuget-6.8?WT.mc_id=DOP-MVP-5002594)
6. [Common NuGet configurations](https://learn.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior?WT.mc_id=DOP-MVP-5002594#config-file-locations-and-uses)
