---
title: "為 ASP.NET Core 建立 Multi-Platform image"
date: 2023-08-22T00:30:00+08:00
lastmod: 2023-08-22T00:30:31+08:00
draft: false
tags: ["docker","dotnet core","aspdotnet core"]
slug: "docker-multi-platform-aspdotnetcore"
---

## 為 ASP.NET Core 建立 Multi-Platform image

最近公司正在逐步汰換使用年限到期的電腦：由 MacBook Pro (intel) 換為 MacBook Pro (M2 Pro)，CPU 架構從 x86/x64 改為 arm64，許多軟體工具都需要改用特定版本才能正常使用，雖然在 M1 初問世就已經做過一輪測試跟 poc 也先做一些預先調整，其中一個就是啟用 `buildx` 來建立 Multi-Platform image，只是當初沒有經過全面驗證，這次實際更換機器時問題就陸續出現，今天就是要來紀錄一下 ASP.NET Core 的 Multi-Platform image 在 arm 上遇到的問題

## 基本環境說明

1. macOS Ventura 13.4
2. OrbStack 0.13.0(1910)
3. docker images
    - mcr.microsoft.com/dotnet/sdk:6.0
    - mcr.microsoft.com/dotnet/aspnet:6.0
    - mcr.microsoft.com/dotnet/sdk:8.0-preview-alpine
    - mcr.microsoft.com/dotnet/aspnet:8.0-preview-alpine
4. ASP.NET Core 預設 web api 專案範本
5. build multiple-platform

    ```bash
    docker buildx build -f Dockerfile --platform linux/amd64,linux/arm64 -t yowko/multiplearch --push . 
    ```

## 設定方式

1. 使用 .NET 8 以前 SDK

    > 主要的 dockerfile 結構是官方的，條件式內容則是參考 [GitHub: f2calv/multi-arch-container-dotnet](https://github.com/f2calv/multi-arch-container-dotnet/blob/main/Dockerfile)

    {{<gist yowko 934417a3229a851b81fb263423befb3d>}}

2. 使用 .NET 8 SDK (尚有相容性問題，詳細問題可以看 `心得`)

    > 這個是參考 Microsoft PM - Richard Lander 的文章：[Improving multi-platform container support](https://devblogs.microsoft.com/dotnet/improving-multiplatform-container-support?WT.mc_id=DOP-MVP-5002594)，dockerfile 也是官方的範例來的 [samples/aspnetapp/Dockerfile.alpine-non-root](https://github.com/dotnet/dotnet-docker/blob/main/samples/aspnetapp/Dockerfile.alpine-non-root)

    {{< gist yowko 0b03db6cf93d49e0c46821e40a494ab8>}}

## 心得

- 目前看到 .NET 8 的改變有兩個部份

  1. 新增 `-a` 參數，是 `-r` 的快捷鍵：`${TARGETOS}/${TARGETARCH}` 等於 `${TARGETPLATFORM}`，SDK 內部應該還有一層 mapping 做 RID 的轉換
  2. publish 時 `-c Release` 已改為預設參數

- ASP.NET Core 針對 Multi-Platform 我自己覺得不同以往的部份

  1. stage 1 的 FROM 有指定 builder 的 image platform
  2. restore 有加上 `-a` 指定特定平台套件
  3. image size 差很多 (8: 109MB，6: 306MB)

- 我參考 Microsoft PM - Richard Lander 的文章：[Improving multi-platform container support](https://devblogs.microsoft.com/dotnet/improving-multiplatform-container-support?WT.mc_id=DOP-MVP-5002594) 嘗試幾種不同方式得到幾個不同錯誤，紀錄一下

  1. 用相同 dockerfile 來 build .NET 6 的專案，出現需要安裝 .NET runtime 的提示訊息

        ![1wrongsdk](https://github.com/yowko/picsbed/assets/3851540/e3caec50-20b8-4d53-a95d-73bd236508a3)

  2. 使用 `mcr.microsoft.com/dotnet/sdk:8.0-preview-alpine` builder ，runtime 改用 `mcr.microsoft.com/dotnet/aspnet:6.0` 出現找不到檔案

        ![2nofile](https://github.com/yowko/picsbed/assets/3851540/2676f08d-8f9a-491b-8d0e-7eedaa487b6d)

  3. 指定 `--self-contained true` 會 build fail

        ![3selfcontained](https://github.com/yowko/picsbed/assets/3851540/eec2e6de-ee4d-455a-a517-21f9af74ba11)

  4. 使用 .NET 8 的條件式來編輯出現找不到檔案

        ![2nofile](https://github.com/yowko/picsbed/assets/3851540/2676f08d-8f9a-491b-8d0e-7eedaa487b6d)

## 參考資訊

1. [docker Multi-platform images](https://docs.docker.com/build/building/multi-platform/)
2. [Enable using docker build --platform switch (easily)](https://github.com/dotnet/dotnet-docker/issues/4388?WT.mc_id=DOP-MVP-5002594)
3. [Improving multi-platform container support](https://devblogs.microsoft.com/dotnet/improving-multiplatform-container-support?WT.mc_id=DOP-MVP-5002594)
4. [samples/aspnetapp/Dockerfile.alpine-non-root](https://github.com/dotnet/dotnet-docker/blob/main/samples/aspnetapp/Dockerfile.alpine-non-root?WT.mc_id=DOP-MVP-5002594)
5. [.NET RID 目錄](https://learn.microsoft.com/zh-tw/dotnet/core/rid-catalog?WT.mc_id=DOP-MVP-5002594)
