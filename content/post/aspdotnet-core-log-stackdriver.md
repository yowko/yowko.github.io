---
title: "在 ASP.NET Core 中將 log 寫至 GCP 的 Stackdriver"
date: 2019-06-22T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","GCP"]
slug: "aspdotnet-core-log-stackdriver"
---

## 在 ASP.NET Core 中將 log 寫至 GCP 的 Stackdriver

之前剛好有個功能在內部環境運作時都一直出現錯誤，經過一輪測試後決定將功能搬至 GCP 的 GKE 上執行來確認問題是不是內部環境設定所造成的

為了可以在 GKE 測試，核心功能程式碼幾乎不需要修改，但 log 部份就不得不做些調整，一來不想要大費周章設定 storage 也不想直接把 log file 擺在 container 內，所以打算透過 GCP 的 Stackdriver 來儲存 log，不僅程式修改幅度小還可以直接使用 UI 來查詢 log，相當方便

至於 Stackdriver ，我沒有深入探討，就功能上看來與 Azure 的 ApplicationInsight 應該是一樣的，但純屬個人猜測

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet package

    - Google.Cloud.Diagnostics.AspNetCore 2.0.1

## 在 GCP 建立憑證

憑證是用來讓與 Stackdriver 進行權限驗證用的

1. API 和服務 --> 憑證

    ![1crendential](https://user-images.githubusercontent.com/3851540/59965713-76812280-9544-11e9-81c4-a68b32bb0d06.png)

2. 建立憑證 --> 服務用戶金鑰

    ![2createcredential](https://user-images.githubusercontent.com/3851540/59965715-76812280-9544-11e9-8448-c2f2b8191a0c.png)

    ![3keycreated](https://user-images.githubusercontent.com/3851540/59965716-76812280-9544-11e9-89b7-88559e4f491c.png)

    ![4created](https://user-images.githubusercontent.com/3851540/59965717-7719b900-9544-11e9-8f06-607e13f08bc2.png)

## application 設定

1. 安裝 NuGet package : `Google.Cloud.Diagnostics.AspNetCore`

    - Package Manager

        ```bash
        Install-Package Google.Cloud.Diagnostics.AspNetCore
        ```

    - .NET CLI

        ```bash
        dotnet add package Google.Cloud.Diagnostics.AspNetCore --version 2.0.1
        ```

2. 將 `Stackdriver` 所需設定加至 `appsettings.json`

    ```json
    {
        "Stackdriver": {
            "ProjectId": "YOUR-GOOGLE-PROJECT-ID",
            "ServiceName": "NAME-OF-YOUR-SERVICE",
            "Version": "VERSION-OF-YOUR-SERVICE"
        }
    }
    ```

    - ProjectId

        > 從 google cloud console 上可以找到

        ![5projectid](https://user-images.githubusercontent.com/3851540/59965718-7719b900-9544-11e9-8453-f9d99b456bdd.png)

    - ServiceName

        > 自訂，會當做 log 的名稱

    - Version

        > 自訂，沒看到哪邊有用到

3. 設定 Stackdriver

    > 將以下設定加至 `Startup` 的 `Configure` 方法中

    ```cs
    // 設定 logging 
    loggerFactory.AddGoogle(configuration.GetValue<string>("Stackdriver:ProjectId"));
    var logger = loggerFactory.CreateLogger(configuration.GetValue<string>("Stackdriver:ServiceName"));

    // 實際寫 log
    logger.LogInformation("測試 log stackdriver");
    ```

## 在執行環境中設定憑證

1. 將 GCP 建立並下載的憑證檔案位置設定給環境變數 `GOOGLE_APPLICATION_CREDENTIALS`

    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS={credential_path}
    ```

2. 未設定錯誤

    - 錯誤訊息

        ```text
        Application startup exception: System.InvalidOperationException: The Application Default Credentials are not available. They are available if running in Google Compute Engine. Otherwise, the environment variable GOOGLE_APPLICATION_CREDENTIALS must be defined pointing to a file defining the credentials. See https://developers.google.com/accounts/docs/application-default-credentials for more information.

        ```

    - 錯誤截圖

        ![6error](https://user-images.githubusercontent.com/3851540/59965719-7719b900-9544-11e9-9695-eaa485455c4e.png)

## 從 GCP 上確認 log

1. 記錄 --> 記錄檢視器

    ![7logviewer](https://user-images.githubusercontent.com/3851540/59965720-7719b900-9544-11e9-9ac8-113cc7489490.png)

2. 資源部份選擇 `通用`

    ![8logdetail](https://user-images.githubusercontent.com/3851540/59965722-77b24f80-9544-11e9-972b-c99b67ab82ac.png)

## 心得

也許是我還沒搞清楚官方文件的使用方式，雖然所有需要的資訊在 GCP 的官方文件上都可以找到，但就是散落在各處，加上各處的內容又一定不一致，做了許多嘗試才真的成功 XD    我猜測狀況應該跟 .NET Core 的一樣：更新快又頻繁，文件來不及跟上，以致於舊版文件還殘留著

這是我第一次使用 GCP 服務，整體說來還算是直覺，功能位置還算好找，不過設定的東西很多，一時之間沒能摸懂，也不影響功能的使用

## 參考資訊

1. [設定 .NET Core 適用的 Stackdriver Debugger](https://cloud.google.com/debugger/docs/setup/dotnet?hl=zh-tw)
2. [為 .NET 設定 Stackdriver](https://cloud.google.com/dotnet/docs/stackdriver?hl=zh-tw)
3. [Writing Application Logs](https://cloud.google.com/appengine/docs/flexible/dotnet/writing-application-logs?hl=zh-tw#writing_application_logs_1)
