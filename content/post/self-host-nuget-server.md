---
title: "自建 NuGet Server"
date: 2017-07-11T22:07:00+08:00
lastmod: 2021-10-26T22:07:18+08:00
draft: false
tags: ["套件","NuGet"]
slug: "self-host-nuget-server"
aliases:
    - /2017/07/self-host-nuget-server.html
---
## 自建 NuGet Server

自從 Visual Studio 2010 第一次使用 NuGet 後，開發生涯就此改變了，簡易的套件安裝及管理方式，讓開發人員不需要在自行保留需要的 dll，需要的 config 也可以透過 NuGet 安裝來加入，大幅地減少開發時繁瑣的工作

但有些公司的資安政策並不允許外部連線，這也造成 NuGet 無法使用，一般的解決方式就是透過共用網路資料夾來分享可能會用到的 dll，只是這樣的方式終究不如 NuGet 來得便利

最近的專案有些共用元件，原本都是透過 dll 參考，但常遇到 dll 有更新時，shared folder 中的 dll 檔案被更新，造成不需要使用新版 dll 的 module 也一併被迫使用新版 dll，無形中造成不少問題。所以打算利用自建內部 NuGet Server 來解決問題，因為 NuGet 不同版本可以共存，至少可以讓不需升級的 module 繼續使用舊版本，而不用被迫升級。

## 建立 Web Application

1. 使用 Installed template 建立專案 --> ASP.NET Web Application(.NET Framework)

    ![1addwebapp](https://user-images.githubusercontent.com/3851540/28053869-51124f72-6645-11e7-83d8-6c2163a41bca.png)

2. 使用 Empty template

    ![2empty](https://user-images.githubusercontent.com/3851540/28053871-51505c54-6645-11e7-846a-f08b41c705e9.png)

## 安裝 NuGet.Server

> 兩種方法擇一即可，安裝完成會異動到下列內容
>
> 1. 加入 `Default.aspx`
> 2. 加入 `favicon.ico`
> 3. 新增 `DataServices` 資料夾並加入 `Packages.svc` 與 `Routes.cs`
> 4. 新增 `Packages` 資料夾並加入 `Readme.txt`
> 5. 修改 `{專案名稱}.csproj`
> 6. 修改 `packages.config`
> 7. 修改 `Web.config`

![2-1change](https://user-images.githubusercontent.com/3851540/28053870-51364198-6645-11e7-9527-d5acb612c02c.png)

1. Package Manager UI
    * 專案 上按右鍵 --> Manage NuGet Packages...

        ![3nabage](https://user-images.githubusercontent.com/3851540/28053876-51b6c19c-6645-11e7-82eb-0031c4dd003b.png)

    * Browse tab --> 搜尋 `NuGet.Server` --> Install

        ![4install](https://user-images.githubusercontent.com/3851540/28053872-515169e6-6645-11e7-8acf-e8290fc0c448.png)

2. Package Manager Console

    * Visual Studio 主選單 Tools --> NuGet Package Manager --> Package Manager Console

        ![5mangeconsole](https://user-images.githubusercontent.com/3851540/28053873-51599bac-6645-11e7-958c-51798f9b382e.png)

    * 執行 `Install-Package NuGet.Server`

        ![6installpackage](https://user-images.githubusercontent.com/3851540/28053874-5159e0f8-6645-11e7-8caa-d0ee5b3746b0.png)

* 如果 ASP.NET Web Application 的 .NET Framework 是 `4.5.2` ， NuGet.Server 需安裝 `2.10.3` 版本，最新版為 2.11.3

## 調整 web.config

1. requireApiKey

    > 設定 push 或是 delete NuGet packages 是否需要 Api Key

2. apiKey

    > 設定 Api Key ，如果 requireApiKey 是 true，這個值就一定得給，否則無法 push package

3. packagesPath

    > 指定 package 儲存目錄，可以是虛擬或是實體目錄，預設是 `~/Packages`

4. allowOverrideExistingPackageOnPush

    > 設定 packages 有相同 `id + 版號` 時是否允許覆蓋。預設是 false ，與 NuGet.org 相同 - 不允許覆蓋

5. ignoreSymbolsPackages

    > 設定是否忽略 symbols packages - `.symbols.nupkg` 與 packages 中包含 `/src` 資料夾的檔案都會被忽略 ，因為 NuGet.Server 沒有包含 symbol server 功能

6. enableDelisting

    > 設定是否以 delist 取代 delete

    * delete 會直接刪除檔案
    * delist 會將檔案設為隱藏，nuget list 時會略過被 delist 的 package，但指定 package 版本安裝時(nuget install packageid -version version)可以正常取得檔案，避免特定版本的相依出現問題

7. enableFrameworkFiltering

    > 設定是否可以依 .NET Framework 來過濾 package

8. aspnet:UseHostHeaderForRequestUrl

    > 如果是 NAT - Network Address Translation 網路環境才需啟用

## 成功安裝

![7result](https://user-images.githubusercontent.com/3851540/28053875-515aa97a-6645-11e7-8b75-736d1568d102.png)

1. 提供加入 NuGet Source Url (供 Visual Studio 加入用)

    > `http://localhost:15069/nuget`

2. Push NuGet package 的 Url (發行 NuGet package 用)

    > `nuget.exe push {package file} {apikey} -Source http://localhost:15069/api/v2/package`

## 心得

安裝上很簡易，缺點是畫面醜醜的，但我們工程師在意的是功能，絕對不會因為畫面難看就嫌棄它的XD

## 參考資訊

1. [NuGet.Server](https://docs.microsoft.com/en-us/nuget/hosting-packages/nuget-server?WT.mc_id=DOP-MVP-5002594)
