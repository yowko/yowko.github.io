---
title: "Jenkins 自動 Publish NuGet Package"
date: 2017-07-20T23:49:00+08:00
lastmod: 2021-11-02T23:23:21+08:00
draft: false
tags: ["套件","Jenkins","NuGet","Visual Studio"]
slug: "jenkins-publish-nuget-package"
aliases:
    - /2017/07/jenkins-publish-nuget-package.html
---
## Jenkins 自動 Publish NuGet Package

公司專案共用元件正在逐步由直接 add dll reference 改為使用 NuGet 管理，所以有不少專案需要打包成 NuGet Package，原本都是透過 NuGet Package Explorer 這個 GUI 軟體來打包並 publish 至內部 NuGet Server，但可以預見如果專案一多，耗費的時間一定相當可觀，加上人為操作難免會出錯，因此打算透過 Git Webhook 自動觸發 Jenkins 進行 CI build，接著再自動 publish 至內部 NuGet server

需求是不是很簡單？！我當時就閃過需求簡單的念頭，直到設定時才發現問題還不少，接著立馬想到難怪以前客戶常覺得簡單，因為就算是身為工程師有時也會錯估技術面所需時間，而客戶根本不知道技術層面可能會遇到的問題，難免造成兩方意見相左

回到重點，就來看看如何設定 Jenkins 來自動 publish NuGet package 吧

## 安裝 NuGet Packager

NuGet Packager 是 Visual Studio 專案範本，得透過 Visual Studio 擴充套件來安裝

1. Visual Studio 主選單 Tools --> Extensions and Updates...

    ![1extension](https://user-images.githubusercontent.com/3851540/28425524-29c42ad4-6da3-11e7-8a1e-7f3cf1cfd6af.png)

2. `Online` tab --> 搜尋 `NuGet Packager` --> Download 進行安裝

    ![2nugetpackager](https://user-images.githubusercontent.com/3851540/28425525-29f35b74-6da3-11e7-8d37-cfd5d58b1b22.png)

    ![3install](https://user-images.githubusercontent.com/3851540/28425526-2a0dfb32-6da3-11e7-8c35-efc1551c59ed.png)

3. 安裝之後要重啟 Visual Studio

    ![4restart](https://user-images.githubusercontent.com/3851540/28425530-2a1c6118-6da3-11e7-8a61-012b781e412f.png)

## 建立 NuGet 專案

1. 使用 NuGet 專案範本建立 NuGet 專案

    > 正確安裝 NuGet Packager 就會出現 NuGet 專案範本 選項

    ![5nugetproj](https://user-images.githubusercontent.com/3851540/28425527-2a11820c-6da3-11e7-9ad1-d9c0ab17f449.png)

2. NuGet 專案結構

    ![6nugetstructure](https://user-images.githubusercontent.com/3851540/28425529-2a1aaa9e-6da3-11e7-9eea-7b6303293efe.png)

    > 如果對於 NuGet 專案結構用途不清楚可以參考 [使用 NuGet Package Explorer 建立 NuGet 套件](/nuget-package-explorer)

## 修改 NuGet 資訊

1. 修改 `Package.nuspec`

    > 如果不知道該怎麼填寫 `.nuspec`(尤其是 dependencies 這個部份)，可以使用 NuGet Package Explorer 填寫後，將檔案內容複製過來即可

2. 將所需的檔案複製至適合的位置

    > 以 demo 例子(只將 dto 包裝成 NuGet package)，只需要在 `lib` 下對應的 .net framework folder 中放 dll 即可，但開發階段只需加入 `.gitkeep` 檔案，確保 folder 會被建立即可，dll 待 Jenkins build success 後才放進來打包

    - 什麼檔案該放哪個資料夾可以參考 [使用 NuGet Package Explorer 建立 NuGet 套件](/nuget-package-explorer)

3. 在 NuGet.config 加入 NuGet server url

    - 在 `packageSources` 區段，指定 NuGet server url

        ```xml
        <packageSources>
            <add key="YowkoTest" value="http://localhost:4958/api/v2/package"/>
        </packageSources>
        ```

    - apikeys 不用填

        > NuGet 使用的 apikey 會利用使用者相關資訊來 hash 加密，如果 jenkins 的執行身份現在電腦登入身份一樣，這邊填了才有用，否則還是會出現 apikey 錯誤的異常

        - 錯誤訊息是 `The parameter is incorrect.`

            ![6-1error](https://user-images.githubusercontent.com/3851540/28425528-2a172374-6da3-11e7-9e62-e009d180f695.png)

## 調整 publish 語法

因為前面指定 apikey 無法使用，仍會有 apikey 無效的錯誤，此時可以透過修改 `NuGetPackage.ps1` 中進行 nuget push 時直接指定 apikey 來解決

1. 194 行(如果行號跟程式對不起來，請搜尋 `push`)

    - 原始程式碼

        ```ps1
        $publishTask = Create-Process .\NuGet.exe ("push " + $_.Name + " -Source " + $url)
        ```

    - 修改後程式碼(`{實際的 apikey}` 請換成正確 apikey)

        > 在最後加上 `-ApiKey` 參數與實際的 apikey

        ```ps1
        $publishTask = Create-Process .\NuGet.exe ("push " + $_.Name + " -Source " + $url + " -ApiKey {實際的 apikey}")
        ```

2. 232 行(如果行號跟程式對不起來，請搜尋 `push`)

    - 原始程式碼

        ```ps1
        $task = Create-Process .\NuGet.exe ("push " + $_.Name + " -Source " + $url
        ```

    - 修改後程式碼(`{實際的 apikey}` 請換成正確 apikey)

        > 在最後加上 `-ApiKey` 參數與實際的 apikey

        ```ps1
        $task = Create-Process .\NuGet.exe ("push " + $_.Name + " -Source " + $url + " -ApiKey {實際的 apikey}")
        ```

<!--## 5. 設定專案相依
將 NuGet 專案設定 depends on 主要專案，這樣 build NuGet 專案時就會預設 build 主要專案了
>![7dependon](https://user-images.githubusercontent.com/3851540/28425531-2a2a0278-6da3-11e7-958a-105ec4f213b2.png)-->

## 設定 Jenkins

原則上就是依照一般 Jenkins build .net 專案的設定，如果不熟悉的可以參考 [如何使用 Jenkins 2 建置 .NET 專案](/jenkins-2-build-dotnet-project)

1. 加入 free style 專案
2. 設定 SCM (Source Code Management)
3. 加入兩個 `Build a Visual Studio project or solution using MSBuild`
    - build 主要專案
        - MSBuild Build File

            > 設定建置主要專案

            ```config
            .\Demo\Demo.csproj
            ```

        - Command Line Arguments

            > 將主要專案的 dll 直接產生至 NuGet 專案的 `lib/{.net framework}` 資料夾中

            ```config
            /p:Configuration=Release /p:Platform="anycpu" /p:OutDir=../NuGet.Packager/lib/net462/
            ```

    - build NuGet 專案

        - MSBuild Build File

            > 設定建置 NuGet 專案

            ```config
            .\NuGet.Packager\NuGet.Packager.csproj
            ```

        - Command Line Arguments

            ```config
            /p:Configuration=Release /p:Platform="anycpu"
            ```

## 建置成功後自動發行為 NuGet 套件

- 發行成功

    ![8published](https://user-images.githubusercontent.com/3851540/28425532-2a3d73c6-6da3-11e7-82fc-a4d3634eb860.png)

- 確認已可使用

    ![9success](https://user-images.githubusercontent.com/3851540/28425533-2a87095a-6da3-11e7-9303-7bb380f618f8.png)

## 心得

說實話本來覺得這個動作應該是很容易的，Jenkins build .net project 而已嘛，結果花了不少時間找合適的 NuGet 包裝套件與解決 apikey 的問題(apikey 錯誤的訊息這麼籠統也浪費不少時間 XD)

過程中，apikey 錯誤時會直接跳出新視窗等待輸入正確 apikey，但在 Jenkins 中沒辦法處理新視窗以致 job 會一直卡在執行中，需要手動停止，嘗試過使用 `Timeout`(指定 push time 時間) 與 `NonInteractive`(不跳出互動視窗) 皆無法解決，這個要特別留意

另外還有個問題：發行 NuGet 套件，需要 {id + 版本} 唯一，如果不唯一套件將不會更新，目前的做法尚未處理到這個問題，也需要特別留意，這個問題會在下篇分享個人解決方式

## 參考資訊

1. [使用 NuGet Package Explorer 建立 NuGet 套件](/nuget-package-explorer)
2. [如何使用 Jenkins 2 建置 .NET 專案](/jenkins-2-build-dotnet-project)
3. [NuGet CLI reference](https://docs.microsoft.com/en-us/nuget/tools/nuget-exe-cli-reference?WT.mc_id=DOP-MVP-5002594)
4. [從Visual Studio發佈NuGet Package的好幫手－NuGet Packager](http://blog.darkthread.net/post-2016-04-28-nuget-packager.aspx)
5. [Use Jenkins to restore and publish packages](https://www.visualstudio.com/en-us/docs/package/build/jenkins)
6. [NuGet Packager](https://marketplace.visualstudio.com/items?itemName=OveAndersen.NuGetPackager)
