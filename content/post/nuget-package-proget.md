---
title: "將 NuGet Package 發行至 ProGet"
date: 2017-09-14T23:50:00+08:00
lastmod: 2021-11-01T23:50:30+08:00
draft: false
tags: ["NuGet","Tools"]
slug: "nuget-package-proget"
aliases:
    - /2017/09/nuget-package-proget.html
---
## 將 NuGet Package 發行至 ProGet

之前 [自建 NuGet Server](/2017/07/self-host-nuget-server.html) 曾經介紹過使用 NuGet.Server 套件自建 NuGet server 來儲存 NuGet package，不過使用 NuGet.Server 套件在整個介面及管理功能上都過於簡陋，不適合當做正式服務，因此最後選擇 ProGet 這個產品來當做 NuGet server，過程中遭遇到問題，特別紀錄一下其中的眉眉角角

## 安裝 ProGet

1. 下載 ProGet

    > ProGet 有免費版跟付費版本差別，但安裝檔是相同的，實際安裝時才需選擇
    >
    > 下載位置 ：[Download ProGet](http://inedo.com/proget/download)

2. 安裝 ProGet

    > 安裝上沒有什麼特別之處，只要依實際情況設定即可

    * 授權宣告

        ![1agreement](https://user-images.githubusercontent.com/3851540/30439318-04c7fb28-99a6-11e7-92e6-5cda8e58870f.png)

    * 選擇版本

        ![2edition](https://user-images.githubusercontent.com/3851540/30439319-04c87b84-99a6-11e7-82cf-425b86a3ef93.png)

    * 註冊

        ![3regit](https://user-images.githubusercontent.com/3851540/30439327-05078798-99a6-11e7-8ee5-de47368914dc.png)

    * 安裝路徑

        ![4installpath](https://user-images.githubusercontent.com/3851540/30439320-04cdedd0-99a6-11e7-8495-c01673f85ff2.png)

    * 資料庫

        ![5db](https://user-images.githubusercontent.com/3851540/30439321-04ddb2d8-99a6-11e7-8868-53b509b1f4b6.png)

    * Host 方式

        ![6host](https://user-images.githubusercontent.com/3851540/30439323-04e8f6e8-99a6-11e7-8f15-576b53d5f0fa.png)

    * 執行身份

        ![7account](https://user-images.githubusercontent.com/3851540/30439324-04effb0a-99a6-11e7-9829-9796aaad1a99.png)

    * 安裝

        ![8sumary](https://user-images.githubusercontent.com/3851540/30439325-04f31c90-99a6-11e7-8012-28597447f08f.png)

## 設定 ProGet

1. 登入 ProGet

    > 預設帳號密碼：`Admin`/`Admin`

    ![9login](https://user-images.githubusercontent.com/3851540/30439326-05043624-99a6-11e7-9089-4be84e6e4b60.png)

2. 建立 Feed

    * Create New Feed

        ![10newfeed](https://user-images.githubusercontent.com/3851540/30439328-05090ff0-99a6-11e7-96c9-b58428021fed.png)

    * Create Feed

        > 選擇 Feed Type 及設定 Feed name

        ![11createfeed](https://user-images.githubusercontent.com/3851540/30439329-051558b4-99a6-11e7-974a-4424ecf07f3d.png)

3. 其他設定

    > 依實際環境調整

    ![12feedproperty](https://user-images.githubusercontent.com/3851540/30439330-051e2c8c-99a6-11e7-84f7-67ab6c1862c8.png)

## 發行 Package

1. Packages --> Add Package

    ![14addpackage](https://user-images.githubusercontent.com/3851540/30439333-052e9de2-99a6-11e7-9a4b-26efa397f27e.png)

2. 選擇發行方式

    ![15publishtype](https://user-images.githubusercontent.com/3851540/30439332-052d7dfe-99a6-11e7-8cfc-34f7ed15c36c.png)

## 使用 NuGet api 發行 Package 的安全性設定

這是本篇文章的重點，特別拉出來另外說明，可參照官方說明 - [API Keys in ProGet](http://inedo.com/support/kb/1112/api-keys-in-proget)

* NuGet api 用法

    ![16nugetapi](https://user-images.githubusercontent.com/3851540/30439334-05315316-99a6-11e7-98f4-59537ac0a399.png)

1. Feed 未設定 NuGet API Key，使用帳密上傳

    > 這是 ProGet 的預設做法，但跟其他 NuGet Server 做法不同

    * 使用 nuget api

    * 語法

        ```cmd
        NuGet.exe push {package} -ApiKey {Account}:{Password} -Source http://{ProGet_Server}/nuget/{Feed_Name}/
        ```

    * 範例

        ![17nugetapipush](https://user-images.githubusercontent.com/3851540/30439336-0547f594-99a6-11e7-8b9a-caec584bdc7c.png)

    * 使用 NuGet Package Explorer

        ![18npepublished](https://user-images.githubusercontent.com/3851540/30439335-0547572e-99a6-11e7-9078-25b58fa08a30.png)

2. Feed 設定 NuGet API Key，需允許匿名上傳

    > 未設定會出現 `403 (Forbidden)` 錯誤

    ![22nugetapi403](https://user-images.githubusercontent.com/3851540/30439312-049c28f4-99a6-11e7-8ef9-d74428944625.png)

    ![23npe403](https://user-images.githubusercontent.com/3851540/30439315-04a109b4-99a6-11e7-9568-79ec36158de6.png)

    * 設定 --> Manage Users & Tasks

        ![19manageuser](https://user-images.githubusercontent.com/3851540/30439313-049c7930-99a6-11e7-9df2-4604be1fb53c.png)

    * Tasks --> Add Permission

        ![20addpermission](https://user-images.githubusercontent.com/3851540/30439311-049c3d8a-99a6-11e7-9853-5632b5727a96.png)

    * 將 `Anonymous` 加至 Principals，並把 `Publish Packages` 加至 Tasks

        ![21addprivilige](https://user-images.githubusercontent.com/3851540/30439314-049ca324-99a6-11e7-9c69-700fd67ccd96.png)

    * 成功上傳

        ![24nugetapiok](https://user-images.githubusercontent.com/3851540/30439316-04a3f46c-99a6-11e7-87a7-9cc60e8609a7.png)

        ![25npeok](https://user-images.githubusercontent.com/3851540/30439317-04c2aff6-99a6-11e7-88e5-9d1525c1f24b.png)

## 心得

ProGet 在使用流程還有不小的改善空間，也缺乏系統化的文件來讓第一次使用的人快速上手，以我為例，明明我已經有其他 NuGet server 的使用經驗，但在使用 ProGet 卻佔不到便宜，在權限問題上卡了好久

說明文件及畫面功能的文字也對不起來，明明是個簡單設定卻一直找不到，畫面與文件上的文字不一致造成不少誤導

## 參考資訊

1. [API Keys in ProGet](http://inedo.com/support/kb/1112/api-keys-in-proget)
