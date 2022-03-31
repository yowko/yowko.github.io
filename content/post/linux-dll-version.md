---
title: "在 Linux 上確認 dll 版本"
date: 2022-04-02T00:30:00+08:00
lastmod: 2022-04-02T00:30:31+08:00
draft: false
tags: ["Linux","csharp","dotnet"]
slug: "linux-dll-version"
---

## 在 Linux 上確認 dll 版本

這是為了解決 ASP.NET Core 3.1 的 amd64 image 無法在 arm 晶片 (M1) 上執行而衍生的問題

我嘗試在 ASP.NET Core 3.1 的 application 中升級一個 NuGet package 以套用在 .NET 5 以後加入的功能，為了確認 NuGet package 有成功升級並包進 image，所以想要在 container 中直接確認 dll 版本

## 基本環境說明

1. Debian GNU/Linux 11 (bullseye)
2. NuGet packages

    - Microsoft.Extensions.Hosting 6.0.1

3. Docker images

    - mcr.microsoft.com/dotnet/sdk:6.0
    - mcr.microsoft.com/dotnet/aspnet:6.0-bullseye-slim-amd64

4. JetBrains Rider 2021.3.3

## 執行方式

1. 安裝套件

    ```bash
    apt update && apt install -y mono-utils
    ```

2. 使用套件來取得 dll 版本

    - 語法

        ```bash
        monodis --assembly {dll file} |grep Version
        ```

    - 範例

        ```bash
        monodis --assembly Microsoft.Extensions.Hosting.dll |grep Version
        ```

        ![1dllversion](https://user-images.githubusercontent.com/3851540/159893854-9f23b444-84dd-40fe-839f-12c3ca790233.png)

## 心得

感覺上使用機會不高，但既然查了資料還是紀錄一下加深印象

如果仔細看可以發現 dll 的版本是 `6.0.0.0` 與 NuGet package 的版本 (`Microsoft.Extensions.Hosting 6.0.1`) 對不上，不過從開發工具來看 `AssemblyVersion` 就是 `6.0.0.0`

![2assemblyversion](https://user-images.githubusercontent.com/3851540/159893862-912e6907-ffd2-43b8-adf1-da1e13aa5cad.png)

## 參考資訊

1. [How to get the AssemblyVersion of a .Net file in Linux](https://stackoverflow.com/a/69212442)
