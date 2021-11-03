---
title: "全新的 Jenkins .NET Build Server 該安裝什麼"
date: 2017-02-07T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps"]
slug: "dotnet-jenkins"
aliases:
    - /2017/02/dotnet-jenkins-2-build-server.html
    - /2017/02/dotnet-jenkins/
---
## 全新的 Jenkins .NET Build Server 該安裝什麼

之前開發 Jenkins Job 都是在個人電腦上，建置完成後要移至新的 build server 時又遭遇一些問題，主因就是 build server 上沒有安裝 visual studio，所以紀錄一下該如何排除

## 文章大綱

1. 安裝 .NET Framework Developer Pack
2. 安裝 NuGet
3. 安裝與設定 msbuild tool
4. 解決找不到 `Microsoft.WebApplication.targets"`

## 安裝 .NET Framework Developer Pack

- [Microsoft .NET Framework 4.6.2 Developer Pack](https://www.microsoft.com/en-us/download/details.aspx?id=53321)
  - 包含對應版本的 .NET Framework
  - 包含對應版本的 .NET Framework targeting Pack 及 SDK

## 安裝 NuGet

![1missnuget](https://cloud.githubusercontent.com/assets/3851540/22176990/a765c910-e050-11e6-9f96-fc45a22c4f60.png)

- 完整安裝
  - 使用 Chocolatey 安裝
    - choco install nuget.commandline
  - 會自動將安裝路徑加至環境變數中
- 僅下載 NuGet 執行檔
  - <https://nuget.org/nuget.exe>
  - 需手動加至環境變數或是直接指定路徑使用

## 安裝與設定 msbuild tool

![2missmsbuild](https://cloud.githubusercontent.com/assets/3851540/22176992/a76651fa-e050-11e6-9fc1-b00808612460.png)

- [Microsoft Build Tools 2015](https://www.microsoft.com/zh-tw/download/details.aspx?id=48159)

    ![3installmsbuild](https://cloud.githubusercontent.com/assets/3851540/22176991/a7662374-e050-11e6-8a2a-77962c0f03ce.png)
- Jenkins 設定 msbuild 位置

    ![4configmsbuild](https://cloud.githubusercontent.com/assets/3851540/22177013/f1ef2616-e050-11e6-9092-e4e89046f6b8.png)

    ![5configmsbuild](https://cloud.githubusercontent.com/assets/3851540/22177017/fdfead96-e050-11e6-8b1b-f130aa269ba9.png)

## 找不到 Microsoft.WebApplication.targets

- 錯誤訊息

    ```log
    Microsoft.WebApplication.targets" was not found ...
    ```

- 錯誤截圖

    ![5webtargets](https://cloud.githubusercontent.com/assets/3851540/22176993/a767631a-e050-11e6-8713-dab7756fee36.png)

- 目前找到的資料方法不外乎兩種
  - 1. 安裝 visual studio
  - 2. 複製需要的檔案及資料夾

- `Microsoft.WebApplication.targets` 檔案已上傳 [GitHub](https://github.com/yowko/Microsoft.WebApplication.targets) , 目前只有 v10.0 及 v14.0

## 參考資訊

1. [A .NET Build Server Without Visual Studio](http://nickberardi.com/a-net-build-server-without-visual-studio/)
2. [How to build .NET 4.6 Framework app without Visual Studio installed?](http://stackoverflow.com/questions/32070490/how-to-build-net-4-6-framework-app-without-visual-studio-installed)
3. [Installing NuGet](https://docs.microsoft.com/zh-tw/nuget/guides/install-nuget?WT.mc_id=DOP-MVP-5002594)
