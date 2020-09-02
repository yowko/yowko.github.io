---
title: "在 macOS 上建立開發用 .NET Core 憑證"
date: 2020-02-23T21:30:00+08:00
lastmod: 2020-02-23T21:30:31+08:00
draft: false
tags: ["macOS","dotnet core"]
slug: "macos-dotnet-cert"
---

## 在 macOS 上建立開發用 .NET Core 憑證

現在的 web 環境幾乎都會要求 https，這樣的改變當然也套用在開發上，像是 HTTP/2 雖然在協定本身允許非加密的 HTTP 協定，但絕大部份的 client 端僅提供 TLS 加密的 HTTP/2 協定實作，這讓 TLS 加密的 HTTP/2 成為了實際上的標準

為了及早處理憑證問題以及程式碼管理，可以在開發階段安裝開發用憑證，簡單紀錄一下語法備用

## 基本環境說明

1. macOS Catalina 10.15.3
2. .NET Core SDK 3.1.102

## 安裝方式

1. 安裝

    ```bash
    dotnet dev-certs https --trust
    ```

    ![1install](https://user-images.githubusercontent.com/3851540/75106610-697b7c80-5659-11ea-8771-71f630d7c22d.png)

2. 移除

    > 記得加上 `sudo`

    ```bash
    dotnet dev-certs https --clean
    ```

    ![3clean](https://user-images.githubusercontent.com/3851540/75106612-6d0f0380-5659-11ea-81ff-088d845b771a.png)

    - 未加 `sudo` 錯誤

        ```txt
        There was an error trying to clean HTTPS development certificates on this machine.
        Removing the requested certificate would modify user trust settings, and has been denied.
        ```

        ![2nosudo](https://user-images.githubusercontent.com/3851540/75106611-6bddd680-5659-11ea-9619-6f02838648ce.png)

## 安裝前後差異

1. 安裝前

    > 出現安全警示，無法直接瀏覽

    ![3errorpage](https://user-images.githubusercontent.com/3851540/75106613-6da79a00-5659-11ea-89d0-726a15522181.png)

    ![4invalidcert](https://user-images.githubusercontent.com/3851540/75106614-6e403080-5659-11ea-94d7-2147e5e92e02.png)

2. 安裝後

    > 無警示提醒

    ![5normalpage](https://user-images.githubusercontent.com/3851540/75106616-6ed8c700-5659-11ea-8083-ee16ae23a72b.png)

    ![6goodcert](https://user-images.githubusercontent.com/3851540/75106617-6ed8c700-5659-11ea-99ee-f46bd0b504db.png)

## 心得

雖然憑證問題不是很難解決，但憑證問題在過去的經驗中，有時候很難釐清該由誰處理，有的團隊是開發人員負責，有的則是 MIS 負責，另外環境 os 因為實作方式不同也會有不同的處理方式，但身為解決問題的人，我想多了解一點總不會是壞事，不過有時候就是想了解也不一定有合適的學習對象

## 參考資訊

1. [HTTP/2](https://zh.wikipedia.org/wiki/HTTP)
2. [Troubleshooting .NET Core Dev Certs on MacOS](https://dev.to/cesarcodes/troubleshooting-net-core-dev-certs-on-macos-179d)
3. [Enforce HTTPS in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-3.1&tabs=visual-studio&WT.mc_id=DOP-MVP-5002594#troubleshoot-certificate-problems)
