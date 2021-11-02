---
title: "關於指定 NuGet 安裝資料夾"
date: 2017-07-29T23:25:00+08:00
lastmod: 2021-11-01T23:25:05+08:00
draft: false
tags: ["NuGet","Visual Studio"]
slug: "nuget-install-folder"
aliases:
    - /2017/07/nuget-install-folder.html
---
## 關於指定 NuGet 安裝資料夾

之前文章 [指定 NuGet packages 存放位置](/nuget-folder)，有介紹如何修改 NuGet 套件的安裝目錄，後來同事測試後有更簡單的做法，因此紀錄一下

使用設定檔的做法相同，但步驟簡易不少，主要目的還是為了避免相同套件被各專案重複下載並佔用無謂的磁碟空間

## 在 Solution 中加入 NuGet.config

透過 Visual Studio 在 Solution 這層加入 NuGet.config 指定 NuGet package 路徑

1. 直接加入 NuGet.config

    * Solution 上按右鍵 --> Add --> New Item...

        ![1addfile](https://user-images.githubusercontent.com/3851540/28745857-ec4b2492-74b3-11e7-94c4-7f4cd220cc7f.png)

    * 搜尋 `xml` --> 使用 `XML File` --> 指定檔名為 `NuGet.config`

        ![2addxml](https://user-images.githubusercontent.com/3851540/28745860-ec99983e-74b3-11e7-916d-0b9d992f8130.png)

    * 修改 `NuGet.config`

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <config>
                <add key="repositoryPath" value="C:\NuGetPackages" />
            </config>
        </configuration>
        ```

    * 加入結果

        ![2-1RESULT](https://user-images.githubusercontent.com/3851540/28745858-ec716b3e-74b3-11e7-90ae-a2c660e604e8.png)

2. 先新增資料夾，再加入 NuGet.config

    > 資料夾名稱僅作為識別用，無實際作用，名稱也沒有限制

    * Solution 上按右鍵 --> Add --> New Solution Folder...

        ![3addfolder](https://user-images.githubusercontent.com/3851540/28745863-ec99b986-74b3-11e7-854e-d55ef8e43d1b.png)

    * 接著加入檔案方式與直接加入 NuGet.config 相同
    * 加入結果

        ![4RESULT](https://user-images.githubusercontent.com/3851540/28745861-ec99b6ac-74b3-11e7-83ea-c04c933fc96c.png)

3. <span style="color:red">重啟修改的專案(必要條件)</span>

## 從檔案系統中加入 NuGet.config

> 在 Solution 的資料夾中加入 NuGet.config

1. 直接加入 NuGet.config
2. 先新增資料夾，再加入 NuGet.config
    * 資料夾名稱  <span style="color:red">必需是 `.nuget`</span>
    * 加入 NuGet.config

3. <span style="color:red">重啟修改的專案(必要條件)</span>

## 兩者功能完全相同

* 僅在 Visual Studio 顯示上有差異
* 實際在檔案系統中兩者行為完全一樣

    > 都會在 `.sln` 同層位置加上 `NuGet.config`，不會額外建立資料夾

* 透過 Restore NuGet Packages 將套件下載至指令位置

    ![5restore](https://user-images.githubusercontent.com/3851540/28745862-ec99c688-74b3-11e7-819c-c7b6d5416b66.png)

## 其他可以設定的位置

1. 全域 config

    > `%APPDATA%\NuGet\NuGet.Config`

    * 可以使用 `nuget -configFile` 來指定不同 config 檔案

2. machine 等級 config
    * NuGet 2.6 to NuGet 3.5

        > `%ProgramData%\NuGet\Config[\{IDE}[\{Version}[\{SKU}\]]]NuGet.Config`

        * {IDE} 是 VisualStudio
        * {Version} 是 Visual Studio 版號(e.g. 14.0
        * {SKU} 是版本 (e.g. Community, Pro, or Enterprise)

    * NuGet 4.0+

        > `%ProgramFiles(x86)%\NuGet\Config`

3. default config
    * NuGet 2.6 to NuGet 3.5

        > `%PROGRAMDATA%\NuGet\NuGetDefaults.Config`

    * NuGet 4.0+

        > `%ProgramFiles(x86)%\NuGet\Config`

## 心得

我實際測試下 <span style="color:red">Visual Studio 2015 與 Visual Studio 2017 行為皆相同，上述幾個方法都適用</span>

行為與官方文件 - [Configuring NuGet behavior](https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior?WT.mc_id=DOP-MVP-5002594) 有落差：官方說 `.nuget` 的用法僅適用於 NuGet 3.3 及以前版本，但我測試 NuGet 4.1.0.2450 一樣可用；幾個預設 config 位置並不好找，檔案也不一定預設存在，不確定是文件上的問題還是行為正在調整

## 參考資訊

1. [指定 NuGet packages 存放位置](/nuget-folder)
2. [Configuring NuGet behavior](https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior?WT.mc_id=DOP-MVP-5002594)
