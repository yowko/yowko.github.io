---
title: "使用 NuGet Package Explorer 建立 NuGet 套件"
date: 2017-07-12T23:11:00+08:00
lastmod: 2018-09-23T23:11:42+08:00
draft: false
tags: ["套件","NuGet","Tools"]
slug: "nuget-package-explorer"
aliases:
    - /2017/07/nuget-package-explorer.html
---
# 使用 NuGet Package Explorer 建立 NuGet 套件
公司許多專案中有不少底層功能是可以共用的(ex. 資料庫相關操作、redis 相關操作、web request、....etc)，原本都是使用 dll 參考，但這樣的方式有些缺點：1. 如果 dll 是放在共用資料庫供所有專案參考，就可能出現 A 專案需要更新 dll，但 B 專案沒有使用新 dll 的需求可以繼續延用舊 dll，但更新共用資料夾中的 dll 檔案時就會強迫所有專案一併更新，這可能造成其他專案因此出現問題 2. 如果各個專案自行管理 dll，雖然能解決無差別更新的問題，只是這樣一來就跟沒有共用一樣了

最後決定將共用的功能都包裝成 NuGet package 讓其他專案使用 NuGet 來加入參考，因為 NuGet 有版本的概念，可以避免無差別更新的狀況，各專案可以依實際狀況來進行版本升級

## 什麼是 NuGet Package Explorer

NuGet Package Explorer 可以讓我們簡單快速地建立 NuGet Package，也可以直接從磁碟或是 NuGet Server 開啟 NuGet Package metadata 檔案(.nupkg)

![1npe](https://user-images.githubusercontent.com/3851540/28123373-68c3733a-6753-11e7-9322-cf7751894b68.png)

![2newtonjson](https://user-images.githubusercontent.com/3851540/28123377-68df3854-6753-11e7-9d4a-34a777b61c02.png)

![3newtonjsondata](https://user-images.githubusercontent.com/3851540/28123375-68ce8b44-6753-11e7-8b7b-00697da17b8f.png)

## 安裝 NuGet Package Explorer

1.  使用 `Chocolatey`

    > `choco install nugetpackageexplorer`

2.  下載安裝檔 - [載點](https://npe.codeplex.com/downloads/get/clickOnce/NuGetPackageExplorer.application)

    > 如果曾經安裝過 3.9 以前版本，需先移除才能再次安裝 (2017/07/12 最新版本為 3.23 )

## 建立 NuGet Package

1.  Create a mew package

    ![4createnewpackage](https://user-images.githubusercontent.com/3851540/28123379-68f0da46-6753-11e7-8cdf-5538efa08628.png)

2.  編輯 Package metadata

    ![5editmetatada](https://user-images.githubusercontent.com/3851540/28123378-68f04090-6753-11e7-93f7-cab6b2c8fa79.png)

    ![27editmetadata](https://user-images.githubusercontent.com/3851540/28124645-e1a2cd2a-6756-11e7-8e6f-3744e68a7fbf.png)

    *   `Id` + `Version` 需要是 unique

    *   結果對應

        ![6metadatamapping](https://user-images.githubusercontent.com/3851540/28123380-6901b3c0-6753-11e7-86ee-a12e73b7fff8.png)

3.  Edit dependencies

    > 用來加入 NuGet 的相依關係

    ![7editdenpendencies](https://user-images.githubusercontent.com/3851540/28123381-69045d1e-6753-11e7-8c39-2183e1c5d0f4.png)

    *   Add a new group

        ![8addagroup](https://user-images.githubusercontent.com/3851540/28123382-69076ec8-6753-11e7-95fa-a2492d0436cb.png)

    *   Target framework 填上目標 .NET Framewrok

        ![9framewrok1](https://user-images.githubusercontent.com/3851540/28123385-691fed4a-6753-11e7-9fab-d09e6deff3b7.png)

    *   按下任一個欄位 Target framework 會自動更新為正確名稱

        ![10framework2](https://user-images.githubusercontent.com/3851540/28123384-691c21f6-6753-11e7-9050-4b889c911531.png)

    *   Select dependency from NuGet feed

        ![11selecenuget](https://user-images.githubusercontent.com/3851540/28123394-697a3c00-6753-11e7-83e3-5a4c99a58014.png)

    *   Add new dependency

        ![12addnewdenpendency](https://user-images.githubusercontent.com/3851540/28123387-69314144-6753-11e7-8fd2-c18d82749521.png)

    *   加入成功

        ![13added](https://user-images.githubusercontent.com/3851540/28123388-69493628-6753-11e7-8e7f-66371daa978f.png)

    *   安裝時會出現相依提示

        ![13-1denpendency](https://user-images.githubusercontent.com/3851540/28123386-692af406-6753-11e7-8a2d-f7a81c75e876.png)

4.  Edit assembly references

    > 用來加入 dll 參考

## 加入資料夾及檔案

在 Package contents 區塊空白處按下滑鼠右鍵可以加入資料夾及檔案

![14addfolder](https://user-images.githubusercontent.com/3851540/28123391-69646d44-6753-11e7-8060-47ee96b938cd.png)

資料夾用法請參考下表

|Folder|Description|Action upon package install|
|--- |--- |--- |
|tools|用來放 Powershell scripts 跟可以從 Package Manager Console 執行的程式|內容會被複製到專案資料夾中，`tools` 資料夾會被加至 `PATH`環境變數|
|lib|用來放 `.dll`,`.xml`,`.pdb`|`.dll` 組件會直接被加入參考，`.xml` 及 `.pdb` 會被複製至專案資料夾中|
|content|用來放其他類型檔案|標案會被複製至專案資料夾下|
|build|用來放 MSBuild 的 `.targets` 與 `.props` 檔|會自動將 MSBuild 行為加至 `.csproj`(NuGet 2.x) 或 `project.lock.json` (NuGet 3.x).|
|src|原始碼|發行位址([https://nuget.smbsrc.net/](https://nuget.smbsrc.net/)))與一般 nuget package 不同，一般安裝時不會動作|



*   依 lib 為例
    *   需要加入 lib 目標平台的資料夾

        ![15addnet](https://user-images.githubusercontent.com/3851540/28123389-6950468e-6753-11e7-9eab-998dc56dfc7d.png)

    *   在目標平台資料夾中加入 `.dll` 及 `.pdb`

        ![16ADDFILE](https://user-images.githubusercontent.com/3851540/28123390-695a9d46-6753-11e7-973e-c297e2f12c77.png)

        ![17addedfiles](https://user-images.githubusercontent.com/3851540/28123392-696c75d4-6753-11e7-81f0-2c3b5ae24c57.png)

## 分析 NuGet Package 設定是否正確

1.  NuGet Package Explorer 主選單 TOOLS --> Analyze Package

    ![18analyze](https://user-images.githubusercontent.com/3851540/28123393-6977dec4-6753-11e7-8e12-9a692e297188.png)

2.  分析結果

    > 沒有出現 issue 就可以發行了

    ![19analyzed](https://user-images.githubusercontent.com/3851540/28123395-6a364bfc-6753-11e7-8c0e-3ed6d91718c6.png)


## 發行 NuGet Package

1.  NuGet Package Explorer 主選單 FILE --> Publish...

    ![20publish](https://user-images.githubusercontent.com/3851540/28123369-6898e0f2-6753-11e7-8ac4-7558bb1924bd.png)

2.  填入 Publish Url 及 Publish Key

    >這邊以自建 NuGet Server 為例

    ![21published](https://user-images.githubusercontent.com/3851540/28123368-68988620-6753-11e7-92b1-6bbc6d8fa7b2.png)

## Visual Studio 加入 NuGet Source

1.  Visual Studio 主選單 Tools --> Options

    ![22vsoption](https://user-images.githubusercontent.com/3851540/28123371-689be068-6753-11e7-835a-be79ab65bf4b.png)

2.  NuGet Package Manager --> Package Sources --> +

    ![23nuget](https://user-images.githubusercontent.com/3851540/28123370-689a56e4-6753-11e7-887e-3201447298a5.png)

3.  填入 Name 與 Source --> Upate

    ![24update](https://user-images.githubusercontent.com/3851540/28123372-68c2225a-6753-11e7-82b6-8d2186fd634e.png)

4.  搜尋時調整 source

    ![25pkgsource](https://user-images.githubusercontent.com/3851540/28123374-68c66888-6753-11e7-97a8-1ffa521ce9c5.png)

5.  搜尋結果

    ![26result](https://user-images.githubusercontent.com/3851540/28123376-68cfb884-6753-11e7-8b0c-aa0b7ebcfbd2.png)

## 心得

NuGet Package Explorer 是個簡便工具，用它來建立 NuGet Package 可以省下不少時間。當然也可以透過 NuGet CLI 來建立 NuGet Package，但使用 NuGet CLI 來建立 package 相對繁瑣許多，如果有興趣可以參考 [Creating NuGet packages](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package?WT.mc_id=DOP-MVP-5002594)

之前建立 NuGet Package 時常常覺得設定很簡單，不用特別記，但幾次下來發現每次要建立時還是要找一下設定，回想一下該做什麼，不過跟 NuGet CLI 比還是方便不少，透過幾個簡單步驟就能擁有自己的 NuGet Package 了

# 參考資訊

1.  [NuGetPackageExplorer/NuGetPackageExplorer](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer)
2.  [Creating NuGet packages](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package?WT.mc_id=DOP-MVP-5002594)
