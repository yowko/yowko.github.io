---
title: "如何在 debug 時使用對應組態設定的 web.config"
date: 2017-04-22T22:24:00+08:00
lastmod: 2021-10-08T22:24:33+08:00
draft: false
tags: ["套件","Visual Studio","web.config"]
slug: "webconfig-transform-when-debug"
aliases:
    - /2017/04/webconfig-transform-when-debug.html
---
## 如何在 debug 時使用對應組態設定的 web.config

自從 web.config transform (可以依不同的組態設定 e.g.: debug、release 設定不同的值) 的功能出現後，在不同環境間的變數處理變得容易許多，雖然有方法可以預覽及比較最後產出的 config 內容，但我們有時需要針對不同的組態設定來進行 debug，此時就會發現切換 Visual Studio 到不同組態(configuration)時，依舊使用預設的 web.config 執行，而非對應的 web.{組態}.config ，這是因為 web.config transform 原始設計就是只在執行 `部署` 時才會觸發 config 轉換的緣故，身為一個~~偷懶~~追求效率的工程師，你一定也不想為了 debug 不同組態的參數就手動修改 web.config，如果設定沒幾個就算了，一旦設定有幾十個，我想還是來看看可以怎麼設定吧

## 基本設定

1. 使用預設 ASP.NET MVC 專案範本
2. 在 web.config 加入一個 appsetting

    ```xml
    <add key="yowko" value="yowko"/
    ```

3. 分別修改不同組態時 web.config 的設定內容

    * `Web.Debug.config`

        ```xml
        <appSettings>
            <add key="yowko" value="Debug" xdt:Transform="Replace" xdt:Locator="Match(key)"/>
        </appSettings>
        ```

    * `Web.Release.config`

        ```xml
        <appSettings>
            <add key="yowko" value="Release" xdt:Transform="Replace" xdt:Locator="Match(key)"/>
        </appSettings>
        ```

4. 修改 HomeController

    ```cs
    public ActionResult Index()
    {
        ViewBag.Message = ConfigurationManager.AppSettings["yowko"];
        return View();
    }
    ```

5. 修改 Index.cshtml

    ```cs
    @{
        ViewBag.Title = "Home Page";
    }
    @ViewBag.Message
    ```

## 實際問題

> 切換不同 configuration 仍顯示預設 web.config 的設定

![11default](https://cloud.githubusercontent.com/assets/3851540/25305229/0fa859d4-27aa-11e7-9944-7e4b82aa5233.png)

## 預覽對應組態的 config 結果

1. 在想要預覽的組態 config 上按右鍵 --> Preview Transform

    ![1preview](https://cloud.githubusercontent.com/assets/3851540/25305071/4531f5fe-27a7-11e7-8659-39ffbd9e6070.png)

2. 預覽結果

    > 分割視窗左邊是原本的 web.config，右邊是轉換後的 web.config，畫面上會用紅色表示被修改前的內容，綠色表示修改後的內容

    ![2previewresult](https://cloud.githubusercontent.com/assets/3851540/25305072/453a9c5e-27a7-11e7-84b0-414dbd4fb178.png)

## 設定 build 即觸發 web.config transform

* 修改 .csproj 檔
    1. Upload Project(卸載專案)

        * 專案 --> 右鍵 --> Upload Project

            ![3unloadproject](https://cloud.githubusercontent.com/assets/3851540/25305073/455d6ab8-27a7-11e7-9cc5-a13dd6e1e07a.png)

    2. Edit .csproj

        * 卸載的專案 --> 右鍵 --> Edit {projectname}.csproj

            ![4editcsproj](https://cloud.githubusercontent.com/assets/3851540/25305074/456fb72c-27a7-11e7-9d8d-9aad2888f467.png)

    3. 在設定結尾 `</Project>` 前加入下列設定

        * VS 2015

            ```xml
            <Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(MSBuildToolsVersion)\WebApplications\Microsoft.WebApplication.targets" />
            <Target Name="BeforeBuild">
                <TransformXml Source="Web.template.config" Transform="Web.$(Configuration).config" Destination="Web.config" />
            </Target>
            ```

        * VS 2017

            ```xml
            <Import Project="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v$(MSBuildToolsVersion)\WebApplications\Microsoft.WebApplication.targets" />
            <Target Name="BeforeBuild">
                <TransformXml Source="Web.template.config" Transform="Web.$(Configuration).config" Destination="Web.config" />
            </Target>
            ```

        * VS 2017 因為安裝輕量化，有些共用元件預設沒有安裝，這個功能便是其一

            ![7vs2017install](https://cloud.githubusercontent.com/assets/3851540/25305068/450725cc-27a7-11e7-9f88-d362f138cee2.png)

    4. 儲存後重新載入專案

        * 卸載的專案 --> 右鍵 --> Reload Project

            ![5reload](https://cloud.githubusercontent.com/assets/3851540/25305075/45792cd0-27a7-11e7-9926-8f37d2f80e7d.png)

    5. 複製 web.config 命名為 web.template.config

        > 以後修改就 web.config 的內容就直接修改 `web.template.config`

    6. build project 就會更新 web.config

        * web.config 開啟狀態會要求重新載入

            ![6reloadfile](https://cloud.githubusercontent.com/assets/3851540/25305076/457f6cc6-27a7-11e7-879e-6a90e62070b7.png)

* 使用套件

  * 安裝 `Configuration Transform`

      1. VS 主選單 Tools --> Extensions and Updates

            ![8installconfig](https://cloud.githubusercontent.com/assets/3851540/25305067/44f91ca2-27a7-11e7-99f7-0d8afc4e89dd.png)

      2. 在 Online tab --> 搜尋 `Configuration Transform` --> 安裝

            ![9instlled](https://cloud.githubusercontent.com/assets/3851540/25305069/452632be-27a7-11e7-9086-670d4c7437cb.png)

    * 實際使用

      * 在想要取得的組態 config 上 --> 右鍵 --> Execute transformation

        * e.g. 取得 release 的 config --> 在 web.release.config 上按右鍵執行


            ![10execute](https://cloud.githubusercontent.com/assets/3851540/25305070/452809d6-27a7-11e7-8d5f-7cc39b0da027.png)

    * <span style="color:red">VS 2017 暫無法使用</span> (2017/04/22)

## 實際效果

* 可以依不同的 configuration 顯示對應設定內容

    ![12result](https://cloud.githubusercontent.com/assets/3851540/25305230/0fad1492-27aa-11e7-83ef-17fcc35fd3ea.png)

## 心得

* 使用 .csproj 檔
  * 優點：
    1. 設定一次就永久有效

  * 缺點：
    1. 多了一個 template 檔案，容易造成新進人員混淆
    2. 每次 build 都會執行，會影響開發節奏

* 使用套件
  * 優點：
    1. 有需要才執行，速度可控制
    2. 程式碼維護上比較直覺

  * 缺點：
    1. 需要安裝套件，套件目前仍無法於 vs2017 中使用

## 參考資訊

1. [Web.config Transformation Syntax for Web Project Deployment Using Visual Studio](https://msdn.microsoft.com/en-us/library/dd465326.aspx)
2. [How to enable transformations on build with Visual Studio](https://gist.github.com/EdCharbeneau/9135216)
3. [Configuration Transform](https://marketplace.visualstudio.com/items?itemName=GolanAvraham.ConfigurationTransform)
