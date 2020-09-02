---
title: "設定 NuGet Package 安裝時自動加入 Config 區段 (For NuGet feed v2)"
date: 2017-07-23T18:16:00+08:00
lastmod: 2020-09-01T18:16:58+08:00
draft: false
tags: ["套件","NuGet","web.config"]
slug: "nuget-add-config-for-v2"
aliases:
    - /2017/07/nuget-add-config-for-v2.html
---
# 設定 NuGet Package 安裝時自動加入 Config 區段 (For NuGet feed v2)
部份 NuGet Package 的功能是相依在 config 的設定上，有些設定是 package 正常運作的基本條件，有些設定則是保留給使用者的彈性調整空間，無論目的為何，將相關設定加入 config 都是必要的。

除此之外 config 的儲存位置也有不同，有些直接放在 web.config 或是 app.config，也有單獨存在的(像是 nlog.config)，決定儲存位置的因素就是：有沒有線上修改的需求。放在 web.config 中很直覺但一修改就會造成 IIS restart 影響線上服務，所以需要在 service online 的時間點進行 config 調整，建議將設定抽離 web.config 檔案

回到 NuGet Package 安裝時自動加入 Config 主題上，想必大家都遇過某個套件很好用，但卻缺了必要的 config 設定區段，一開始根本無法使用，上網查該加入哪些設定幾乎可以加入安裝 SOP 中了，原本可以更完美的套件使用體驗不免扣了分。

就來看看如何設定 NuGet Package 安裝時自動加入 Config 區段吧

## 基本設定

1.  專案範本

    > 以下將會使用 Common.Logging 當做範例,示範如何在安裝時自動將相關設定加入 config 中，關於 Common.Loggin 使用，請參考 [使用 Common.Logging 搭配 NLog 及 Log4Net](https://blog.yowko.com/2017/07/common-logging.html)

2.  使用 NuGet Package Explorer

    > 利用 NuGet Package Explorer 來示範打包 NuGet package 讓設定的步驟可以更清楚，NuGet Package Explorer 的詳細用法可以參考 [使用 NuGet Package Explorer 建立 NuGet 套件](https://blog.yowko.com/2017/07/nuget-package-explorer.html)

3.  NuGet Package 將使用自建 NuGet server 進行示範

    > 關於自建 NuGet server 詳細資訊可以參考 [自建 NuGet Server](https://blog.yowko.com/2017/07/self-host-nuget-server.html)

## 建立 NuGet

[使用 NuGet Package Explorer 建立 NuGet 套件](https://blog.yowko.com/2017/07/nuget-package-explorer.html) 有從頭建立的詳細步驟，以下我使用線上的 package 來建立

1.  Open a package from online feed

    ![1createfromfeed](https://user-images.githubusercontent.com/3851540/28498520-d99438c0-6fd1-11e7-8495-06393d8be8fd.png)

2.  指定 Package source --> 搜尋 `common.logging` --> 選擇正確版本(預設列出最新版，可以點選 `show all versions` 找到想要的版本)

    ![2searchcommon](https://user-images.githubusercontent.com/3851540/28498518-d99277a6-6fd1-11e7-8246-606a35b1c629.png)

3.  線上 package 內容

    ![3packagenuspec](https://user-images.githubusercontent.com/3851540/28498519-d992e358-6fd1-11e7-8596-2ef0a3e1d566.png)

## 修改 Package 內容

1.  加入 content 資料夾

    ![4contentfolder](https://user-images.githubusercontent.com/3851540/28498521-d9bb10f8-6fd1-11e7-95b2-99421b9e5104.png)

2.  加入 web.config.transform 檔案

    > 這邊檔案類型與 NuGet server 使用版本有關，請注意，詳細原因請參考文末注意事項

    ![5newfile](https://user-images.githubusercontent.com/3851540/28498522-d9c98200-6fd1-11e7-98d7-ac3479c86e51.png)

    ![6addfile](https://user-images.githubusercontent.com/3851540/28498523-d9cb0a6c-6fd1-11e7-8f8f-fb99dc083940.png)

3.  修改 web.config.transform 內容

    > 新增想要加入 web.config 的內容

    ```xml
    <configuration>
    <configSections>
        <sectionGroup name="common">
            <section name="logging" type="Common.Logging.ConfigurationSectionHandler, Common.Logging"/>
        </sectionGroup>
        <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog"/>
    </configSections>
    <common>
        <logging>
            <factoryAdapter type="Common.Logging.NLog.NLogLoggerFactoryAdapter, Common.Logging.NLog41">
                <arg key="configType" value="INLINE" />
            </factoryAdapter>
        </logging>
    </common>
    <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd" autoReload="true" throwExceptions="false" internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
        <targets async="true">
            <target xsi:type="File" name="f" fileName="${basedir}/logs-inline/${shortdate}.log" layout="${longdate} ${uppercase:${level}} ${message}" />
        </targets>
        <rules>
            <logger name="*" minlevel="Debug" writeTo="f" />
        </rules>
    </nlog>
    </configuration>
    ```

    ![7filecontent](https://user-images.githubusercontent.com/3851540/28498524-d9cb0f30-6fd1-11e7-96c2-b40ab3bbaee9.png)

## 注意事項

1.  上述範例是以 web 專案為例，如果是 windows 類型程式請將檔案改為 `app.config.transform`
2.  如果是 NuGet 2.6 以後版本，需要使用 `.xdt` 轉換

    > 上述範例使用 NuGet.Server，是基於 NuGet feed v2 ，所以需用 `.transform`

    *   NuGet feed v3 (VS 2015 and later / NuGet v3.x and above)

        > [https://api.nuget.org/v3/index.json](https://api.nuget.org/v3/index.json)

    *   NuGet feed v2 (VS 2013 and earlier / NuGet 2.x)

        > [https://www.nuget.org/api/v2](https://www.nuget.org/api/v2)

3.  實測下如果版號未更新，nuspec 的內容會被 cache，會造成 config 的轉換用到舊版

    > 這不確定是不是 NuGet.Server 的問題，嘗試 refresh cache 也沒有效果

4.  uninstall package 時會自動逆向移除加入的內容

    > `.xdt` 則需要自行處理移除內容

5.  修改 config 時，格式會亂掉，需要自行重新排版

    ![8unformat](https://user-images.githubusercontent.com/3851540/28498525-d9dc7a86-6fd1-11e7-9369-569287db7c68.png)

## 心得

實際內容跟步驟都不多，但 NuGet 的版本問題造成搜尋時的作法分歧，花了不少時間才釐清正確的做法

反覆測試後才找出可能是 bug 的問題，還是個人設定錯誤造成的，但總歸一句還是自己學藝不精造成的，還好透過這次機會有搞清楚一些細節

# 參考資訊

1.  [使用 Common.Logging 搭配 NLog 及 Log4Net](https://blog.yowko.com/2017/07/common-logging.html)
2.  [使用 NuGet Package Explorer 建立 NuGet 套件](https://blog.yowko.com/2017/07/nuget-package-explorer.html)
3.  [自建 NuGet Server](https://blog.yowko.com/2017/07/self-host-nuget-server.html)
4.  [Transforming source code and configuration files](https://docs.microsoft.com/en-us/nuget/create-packages/source-and-config-file-transformations?WT.mc_id=DOP-MVP-5002594)
5.  [NuGet](https://www.nuget.org/)
