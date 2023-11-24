---
title: "停用 C# 編譯時特定的警告"
date: 2023-11-24T00:30:00+08:00
lastmod: 2023-11-24T00:30:31+08:00
draft: false
tags: ["dotnet8","dotnet","NuGet","csharp"]
slug: "csharp-disable-warn"
---

## 停用 C# 編譯時特定的警告

之前筆記 [NuGet restore error NU1803](/nuget-restore-nu1803) 紀錄了在某次 build fail 後意外發現 Microsoft NuGet team 的 [HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594) 計劃，接著筆記 [NuGet 設定 Insecure HTTP source](/nuget-insecure) 是根據 Microsoft NuGet team 的新計劃：[HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594) 嘗試了從 NuGet 6.8 開始允許啟用 Insecure NuGet source，但 macOS 上似乎還是有問題實測下仍會拋出 NU1803 Warn，所以這次筆記來記錄如何停用 C# 編譯時特定的警告。

## 基本環境說明

1. macOS Sonoma 14.1.1 (Apple M2 Pro)
2. .NET SDK 8.0.100
3. NuGet 6.8.0.122

    > `dotnet nuget --version`

4. JetBrains Rider 2023.2.3
5. 透過 nexus 來模擬 NuGet HTTP source

    > `docker run -d -p 8081:8081 --name nexus sonatype/nexus3`

## 設定方式：設定專案 `NoWarn` 屬性

1. 方法一：修改 `.csproj`

    ```xml
    <NoWarn>$(NoWarn);NU1803</NoWarn>
    ```

    {{< gist yowko fd2a79d69c8fe662cecf5e9974fdf351 >}}

2. 方法二：build 時加上 `/nowarn:NU1803`

    ```bash
    dotnet build /nowarn:NU1803
    ```

    ![3cli](https://github.com/yowko/picsbed/assets/3851540/f3275383-32a2-4a7b-878c-4f99f24e692f)

## 心得

官網上中英文的說明因為翻譯造成意思不同很常見，但直接少一段是怎麼回事XD

- 英文：[C# Compiler Options to report errors and warnings](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/errors-warnings?WT.mc_id=DOP-MVP-5002594#nowarn)

    ![1en](https://github.com/yowko/picsbed/assets/3851540/a18a93c8-8e99-4901-abb6-ba0bb271e1cb)

- 中文：[用於報告錯誤和警告的 C# 編譯器選項](https://learn.microsoft.com/zh-tw/dotnet/csharp/language-reference/compiler-options/errors-warnings?WT.mc_id=DOP-MVP-5002594#nowarn)

    ![2zh](https://github.com/yowko/picsbed/assets/3851540/060ad4a7-528f-4f71-b949-e30708070c78)

## 參考資料

1. [NuGet restore error NU1803](/nuget-restore-nu1803)
2. [HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594)
3. [NuGet 設定 Insecure HTTP source](/nuget-insecure)
4. [HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594)
5. [[DCR]: HTTPS transition warning NU1803 cannot be suppressed](https://github.com/NuGet/Home/issues/12013?WT.mc_id=DOP-MVP-5002594#issuecomment-1210246326)
6. [C# Compiler Options to report errors and warnings](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/errors-warnings?WT.mc_id=DOP-MVP-5002594#nowarn)
7. [用於報告錯誤和警告的 C# 編譯器選項](https://learn.microsoft.com/zh-tw/dotnet/csharp/language-reference/compiler-options/errors-warnings?WT.mc_id=DOP-MVP-5002594#nowarn)
