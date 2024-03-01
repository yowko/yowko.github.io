---
title: "在不同 mac 上共享 .NET 開發用憑證"
date: 2024-04-02T00:30:00+08:00
lastmod: 2024-04-02T00:30:31+08:00
draft: false
tags: ["dotnet","macOS","tls"]
slug: "share-dotnet-certificate-on-mac"
---

## 在不同 mac 上共享 .NET 開發用憑證

最近團隊為了避免部份功能在開發階段因為憑證問題無法正常運作而造成開發與實際的 production code 有所差異，因此想要逐步套用 https everywhere 的機制。
在 local 開發環境，使用自簽憑證就很方便了，但與其要教會所有人自簽憑證所以就想讓大家在執行團隊環境設定時一併安裝統一的憑證，以免去可能遇到的異常狀況，所以今天就來記錄一下如何在不同的 mac 上共享 .NET 開發用憑證。

## 基本環境說明

1. macOS Sonoma 14.3.1 (Apple M2 Pro)
2. .NET 8.0.101

## 設定方式

1. 使用 .NET SDK 產生憑證

    - 語法

        ```bash
        dotnet dev-certs https 
        [-c|--check] [--clean] [-ep|--export-path <PATH>]
        [--format] [-i|--import] [-np|--no-password]
        [-p|--password] [-q|--quiet] [-t|--trust]
        [-v|--verbose] [--version]
        ```

    - 範例

        ```bash
        dotnet dev-certs https -ep test.pfx -p pass.123
        ```

2. 嘗試直接使用 .NET SDK 匯入憑證失敗

    - 匯入語法

        ```bash
        dotnet dev-certs https --clean -i ~/Downloads/test.pfx -p pass.123
        ```

    - 錯誤訊息

        ```log
        The provided certificate file '/Users/yowko.tsai/Downloads/test.pfx' is not a valid PFX file or the password is incorrect.
        ```

    - 錯誤截圖

        ![1dotnetimport](https://github.com/yowko/picsbed/assets/3851540/2e7d583b-2942-4f66-a862-536ce8eb88db)

3. 將共享憑證匯入到 mac

    - 語法

        ```bash
        security import {pfx 檔案位置} -k ~/Library/Keychains/login.keychain-db -t cert -f pkcs12 -P {pfx 密碼} -A
        ```

    - 範例

        ```bash
        security import ~/Downloads/test.pfx -k ~/Library/Keychains/login.keychain-db -t cert -f pkcs12 -P pass.123 -A
        ```

        ![2securityimport](https://github.com/yowko/picsbed/assets/3851540/dd04d6b6-c272-4937-965e-a9c953833011)

4. .NET SDK 信任憑證

    ```bash
    dotnet dev-certs https --trust
    ```

    ![3dotnettrust](https://github.com/yowko/picsbed/assets/3851540/1a0fb9b4-fddc-437b-a7ee-695d0069e8dd)

5. 確認憑證

    ```bash
    dotnet dev-certs https --check
    ```

    ![4checkcert](https://github.com/yowko/picsbed/assets/3851540/3f26967f-7fd5-4484-a7d5-f84c245731c7)

## 心得

1. 憑證效期只能有一年
2. .NET 5 開始可以使用 pem，但 .NET SDK 匯入還是只支援 pfx，但實際上我也沒有直接使用 .NET SDK 成功匯入過

    ![5pemerror](https://github.com/yowko/picsbed/assets/3851540/561ee9ce-6ac0-4f9d-980b-80557623f52f)

## 參考資訊

1. [Microsoft Learn:Enforce HTTPS in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-8.0&WT.mc_id=DOP-MVP-5002594)
2. [Microsoft Learn:dotnet dev-certs](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-dev-certs?WT.mc_id=DOP-MVP-5002594)
3. [How to fix "The provided certificate file is not a valid PFX file" with dotnet dev-certs https import on macOS](https://olegtarasov.me/dotnet-dev-certs-import-macos/)
4. [Setting up ASP.NET Core dev certs for both WSL and Windows](https://www.fearofoblivion.com/setting-up-asp-net-dev-certs-for-both-wsl-and-windows?ref=olegtarasov.me)
