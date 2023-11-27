---
title: "NuGet restore error NU1803"
date: 2022-08-26T00:30:00+08:00
lastmod: 2023-11-26T00:30:31+08:00
draft: false
tags: ["dotnet6","dotnet","NuGet"]
slug: "nuget-restore-nu1803"
---

## NuGet restore error NU1803

2023/11/26 update:  Microsoft NuGet team 的新計劃：[HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594)，筆記可以參考 [NuGet 設定 Insecure HTTP source](/nuget-insecure) 或是 [停用 C# 編譯時特定的警告](/csharp-disable-warn)

原本團隊使用 `.NET SDK 6.0.201` ，考慮到近期幾個更新可能有助於產品減少錯誤與增進效能，所以團隊便著手相應的升級動作，因為產品建立初期，團隊就已經決定使用 container 技術，因此在升級過程中並沒有遇到什麼困難：只要把 base image 改成新版 (`.NET SDK 6.0.400` 與 `.NET Runtime 6.0.8`) 整個升級就算完成了，跟過去逐一升級 server 的做法相比省事不少

升級完成後一周，同事反應某個專案 build fail，原本都已經 revert change 了，經過一番抽絲撥繭後釐清了問題，快速紀錄一下過程

## 基本環境說明

1. macOS Monterey 12.5.1
2. .NET SDK 6.0.400
3. .NET Runtime 6.0.8
4. NuGet 5.9.0.7134|6.3.0.128

## 錯誤訊息與截圖

1. 錯誤訊息

    ```txt
    [36mtest-target_1        | [0m /src/src/xxxxxxxxx.Common.CacheData/xxxxxxxxx.Common.CacheData.csproj : error NU1803: You are running the 'restore' operation with an 'HTTP' source, 'http://team.sb.xxxxxxxxx.com:8081/repository/nuget-group/'. Non-HTTPS access will be removed in a future version. Consider migrating to an 'HTTPS' source. [/src/xxxxxxxxx.Common.sln]
    [36mtest-target_1        | [0m   Failed to restore /src/src/xxxxxxxxx.Common.CacheData/xxxxxxxxx.Common.CacheData.csproj (in 1.39 sec).
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/186871454-4943ef16-f619-4761-8663-f5a4f5a74f57.png)

## Troubleshoot 過程

1. 疑點一：同個 build job 其他 project 都 build success

    - 推測是該 project 有特殊設定或是安裝特殊套件造成的
    - 檢查 `.csproj` 檔案發現 `TreatWarningsAsErrors` 設定為 true

        ![2treatwarningaserror](https://user-images.githubusercontent.com/3851540/186871490-5493271f-0216-4b93-901f-c61e9a8bda81.png)

2. 疑點二：這個設定是在 2019 加上的，為什麼現在才有問題

    - [NuGet HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594)

        經過 google 發現了 Microsoft 在 2022/08/09 發表了[NuGet HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594) 的計劃，其中重點如下

        - 在 [NuGet 6.3](https://docs.microsoft.com/nuget/release-notes/nuget-6.3?WT.mc_id=DOP-MVP-5002594) 開始導入 [NuGet Warning NU1803](https://docs.microsoft.com/en-us/nuget/reference/errors-and-warnings/nu1803?WT.mc_id=DOP-MVP-5002594) 用來提示使用了非 HTTPS 的 NuGet 資源
        - `2023/11` 開始會將 `NU1803` (使用了非 HTTPS 的 NuGet 資源) 從 warning 改為 error，但允許設定忽略
        - `2024/11` 開始 `NU1803` (使用了非 HTTPS 的 NuGet 資源) 的 error 將無法設定忽略

    - [NuGet 6.3](https://docs.microsoft.com/nuget/release-notes/nuget-6.3?WT.mc_id=DOP-MVP-5002594) 會內建在 `.NET SDK 6.0.400` 中

        ![3sdk604](https://user-images.githubusercontent.com/3851540/186871496-5d9204d6-fd53-4de0-8cb5-d2ddfaad598a.png)

3. 疑點三：local build 沒有出現 NU1803 的 warning

    - 雖然已安裝 .NET SDK 6.0.400 但 NuGet 仍是舊版

        > 我沒有花時間去找原因，不知道是不是我透過 brew 升級 .NET SDK 的關係還是 macOS 的關係

        ![4nuget59](https://user-images.githubusercontent.com/3851540/186871498-3a7b86a0-633e-4ab7-b43e-df22c69ad600.png)

    - nuget cli 無法升級至 NuGet 6.3

        > 2022/08/26 只能升到 NuGet 6.2.1

        ```bash
        sudo nuget update -self
        ```

        ![5nuget62](https://user-images.githubusercontent.com/3851540/186871501-8b41c775-95e9-4f85-b6c0-15e900f89370.png)

## 心得

近期都在忙 SRE 相關工作，好一陣子沒碰 .NET 生態系，這次遇到的問題也算是跨了兩個領域：DevOps 跟 .NET，雖然是 CI/CD 過程的問題，但根本原因還是 .NET 生態系造成的，每次遇到這類問題都能展現 DevOps 跟 SRE 的價值，不過回頭重新檢查似乎也沒那麼偉大XD

2023/11/26 update:  Microsoft NuGet team 的新計劃：[HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594)，筆記可以參考 [NuGet 設定 Insecure HTTP source](/nuget-insecure) 或是 [停用 C# 編譯時特定的警告](/csharp-disable-warn)

## 參考資訊

1. [NuGet Warning NU1803](https://docs.microsoft.com/en-us/nuget/reference/errors-and-warnings/nu1803?WT.mc_id=DOP-MVP-5002594)
2. [HTTPS everywhere](https://devblogs.microsoft.com/nuget/https-everywhere?WT.mc_id=DOP-MVP-5002594)
3. [NuGet 6.3](https://docs.microsoft.com/nuget/release-notes/nuget-6.3?WT.mc_id=DOP-MVP-5002594)
4. [HTTPS Everywhere Update](https://devblogs.microsoft.com/nuget/https-everywhere-update/?WT.mc_id=DOP-MVP-5002594)
5. [NuGet 設定 Insecure HTTP source](/nuget-insecure)
6. [停用 C# 編譯時特定的警告](/csharp-disable-warn)
