---
title: "從 arm64 macOS 建立 Linux x64 ASP.NET Docker Image 卡在 dotnet restore"
date: 2024-04-04T00:30:00+08:00
lastmod: 2024-04-04T00:30:31+08:00
draft: false
tags: ["dotnet","macOS","docker","arm64"]
slug: "arm-docker-build-stuck-on-dotnet-restore"
---

## 從 arm64 macOS 建立 Linux x64 ASP.NET Docker Image 卡在 dotnet restore

之前筆記 [為 ASP.NET Core 建立 Multi-Platform image](/docker-multi-platform-aspdotnetcore/) 紀錄到如何使用 buildx 來建立 linux/amd64 與 linux/arm64 的 docker image，當時提到 .NET 8 以前需要使用條件式編譯，而 .NET 8 開始加強了 container 相關支援在 DOCKERFILE 上會簡潔許多，但當時因為 .NET 8 尚未正式上市，部份功能測試起來不符預期(ex: self-contained deployments)，最近因為團隊開始正式升級 .NET 8，過程中遇到一些問題，今天就來紀錄一下使用 arm (Apple M2 Pro) build linux/amd64 docker image 會卡在 dotnet restore 的狀況

## 基本環境說明

1. macOS Sonoma 14.3.1 (Apple M2 Pro)
2. OrbStack Version 1.4.3 (16673)
3. .NET 8.0.101
4. JetBrains Rider 2023.3.3
5. 預設 ASP.NET Core Web Application Web API 專案範本 (啟用 Linux Docker Support)

    - Dockerfile

        {{<gist yowko c1ca3e171df1f79ba0c5587b28c56a4c "OldDockerfile">}}

    - .dockerignore

        {{<gist yowko c1ca3e171df1f79ba0c5587b28c56a4c ".dockerignore">}}

6. Container Images

    - mcr.microsoft.com/dotnet/aspnet:8.0
    - mcr.microsoft.com/dotnet/sdk:8.0

## 問題描述

1. 預設的 Dockerfile 我自己執行上有遇到問題，稍微調整一下結構

    - Dockerfile

        > 1. 把 `COPY ["Dotnet8ContainerBuild/Dotnet8ContainerBuild.csproj", "Dotnet8ContainerBuild/"]` 改為 `COPY ["Dotnet8ContainerBuild.csproj", "Dotnet8ContainerBuild/"]`
        > 2. `WORKDIR "/src/Dotnet8ContainerBuild"` 與 `COPY . .` 交換順序

        {{<gist yowko c1ca3e171df1f79ba0c5587b28c56a4c "NewDockerfile">}}

2. linux/arm64 build 正常

    - 語法

        ```bash
        docker buildx build -f Dockerfile --platform linux/arm64 -t yowko/dotnet8container:v1 --load . 
        ```

    - 截圖

        ![1arm64](https://github.com/yowko/picsbed/assets/3851540/fdf7b431-e1db-4ca6-b700-b470df5fc45b)

3. linux/amd64 build 卡在 dotnet restore

    - 語法

        ```bash
        docker buildx build -f Dockerfile --platform linux/amd64 -t yowko/dotnet8container:v1 --load . 
        ```

    - 截圖

        ![2amd64](https://github.com/yowko/picsbed/assets/3851540/31145dca-b6b1-4767-b0dc-df3623485895)

## 設定方式

Github 上也不少人反應相關問題：[.NET6 docker build stuck on dotnet restore](https://github.com/dotnet/dotnet-docker/issues/3338)、[Emulated x86 and x64 .NET 7 on ARM64 can't create new project](https://github.com/dotnet/runtime/issues/71856)、[.NET8 docker build stuck on dotnet restore](https://github.com/dotnet/runtime/issues/97828)，Microsoft PM - Richard Lander 的回覆：[GitHub:.NET8 docker build stuck on dotnet restore](https://github.com/dotnet/runtime/issues/97828?WT.mc_id=DOP-MVP-5002594#issuecomment-1921906638) 有兩種解法：

1. Dockerfile 中使用 `$BUILDPLATFORM`

    > 詳細內容可以參考 Richard Lander 的文章 [Improving multi-platform container support](https://devblogs.microsoft.com/dotnet/improving-multiplatform-container-support/?WT.mc_id=DOP-MVP-5002594)

    - 在 build 時使用 `--build-arg BUILDPLATFORM` 來指定平台

        ```yaml
        FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build 
        ```

    - 修改後 Dockerfile

        {{<gist yowko c1ca3e171df1f79ba0c5587b28c56a4c "Dockerfile">}}

    - 實際效果

        ![3good](https://github.com/yowko/picsbed/assets/3851540/c2b82e0e-2eed-4c0b-a88a-dccdb42efa8a)

2. 使用 `Rosetta` 模擬器

    > `Rosetta` 在 M1 問市時測試過效能不佳，後來就沒有再使用

## 心得

目前這個 issue： [GitHub:.NET8 docker build stuck on dotnet restore](https://github.com/dotnet/runtime/issues/97828?WT.mc_id=DOP-MVP-5002594) 還沒有解決，如果需要在 arm 上建立 linux/amd64 docker image，目前使用 `$BUILDPLATFORM` 來指定平台是相對方便的，雖然這樣的做法會讓 Dockerfile 變髒，但至少可以解決問題

完整程式碼請參考：[Github:yowko/Dotnet8ContainerBuild](https://github.com/yowko/Dotnet8ContainerBuild)

## 參考資訊

1. [為 ASP.NET Core 建立 Multi-Platform image](/docker-multi-platform-aspdotnetcore/)
2. [GitHub:.NET8 docker build stuck on dotnet restore](https://github.com/dotnet/runtime/issues/97828?WT.mc_id=DOP-MVP-5002594#issuecomment-1921906638)
3. [Improving multi-platform container support](https://devblogs.microsoft.com/dotnet/improving-multiplatform-container-support/?WT.mc_id=DOP-MVP-5002594)
4. [.NET6 docker build stuck on dotnet restore](https://github.com/dotnet/dotnet-docker/issues/3338)
5. [Emulated x86 and x64 .NET 7 on ARM64 can't create new project](https://github.com/dotnet/runtime/issues/71856)
6. [Github:yowko/Dotnet8ContainerBuild](https://github.com/yowko/Dotnet8ContainerBuild)
