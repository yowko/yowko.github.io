---
title: "專案間如何共用 config 設定 - 使用 MSBuild Community Tasks"
date: 2017-08-13T23:28:00+08:00
lastmod: 2020-09-01T23:28:38+08:00
draft: false
tags: ["套件","MSBuild"]
slug: "config-msbuild-community-tasks"
aliases:
    - /2017/08/config-msbuild-community-tasks.html
---
# 專案間如何共用 config 設定 - 使用 MSBuild Community Tasks
之前文章 [專案間如何共用 config 設定 - 使用 MSBuildExtensionPack](https://blog.yowko.com/2017/08/shared-config.html) 提供跨專案使用相同 config 設定值的概念，也介紹了透過 MSBuildExtensionPack 實作的細節

今天則要介紹另一套功能類似的 library - MSBuild Community Tasks，在進行功能比較前，先來看看該如何使用吧

## 安裝 MSBuild Community Tasks

*   [MSBuild Community Tasks 下載位置](https://github.com/loresoft/msbuildtasks/releases)


1.  使用 `.msi` 安裝

    ![1install](https://user-images.githubusercontent.com/3851540/29250918-e9089f64-807d-11e7-8dbd-2d92f6081f7e.png)

    *   如果沒有與 Visual Studio 整合的需求 可以移除該功能

        ![2removcvs2010](https://user-images.githubusercontent.com/3851540/29250919-e92edc6a-807d-11e7-8497-9e026aee0005.png)

    *   無法改變安裝路徑

        ![3nochangepath](https://user-images.githubusercontent.com/3851540/29250920-e94c7446-807d-11e7-83da-021dae9080d6.png)

        > 預設安裝路徑是 `C:\Program Files (x86)\MSBuild\MSBuildCommunityTasks`

2.  使用 `.zip`

    > 將 `.zip` 解壓縮至想要位置即可

## 在專案資料夾中新增 config template

1.  複製一份 config 並 rename 成其他名稱
    *   範例：

        > `Web.config.template`

2.  將 config 內容想要由共用設定決定的部份改成變數
    *   範例：
        *   在 appSettings 區段中加上一個 `TestGlobalSetting`
        *   value 則使用變數 `$YowkoTest`

    *   完整內容

        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <configuration>
            <appSettings>
                <add key="TestGlobalSetting" value="$YowkoTest" />
            </appSettings>
        </configuration>
        ```

## 在共用資料夾中新增共用的參數設定檔

*   檔案不需要跟特定專案放在一起，方便與其他專案共用
*   關於 element 的詳細說明可以參考 [MSBuild Properties](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-properties?WT.mc_id=DOP-MVP-5002594)


1.  使用 xml 格式

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    ```

2.  以 `Project` 為 root element

    ```xml
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    </Project>
    ```

3.  加入 `PropertyGroup` element

    ```xml
    <PropertyGroup>
    </PropertyGroup>
    ```

4.  將目標參數值以參數 element 包覆

    ```xml
    <YowkoTest>ValueChanged</YowkoTest>
    ```

5.  完整內容

    ```xml
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <PropertyGroup>
            <YowkoTest>ValueChanged</YowkoTest>
        </PropertyGroup> 
    </Project>
    ```

6.  範例檔名：`properties.xml`；路徑：`C:\properties.xml`


## 在專案資料夾中新增 msbuild 定義檔

*   用來執行 config 內容取代作業


1.  使用 xml 格式

2.  以 `Project` 為 root element 並指定 `DefaultTargets`

    *   DefaultTargets

        > 宣告預設執行的 `Target` 內容，稍後會定義，名稱可自訂

    *   範例：

        ```xml
        <Project DefaultTargets="Default" xmlns="http://schemas.microsoft.com/developer/msbuild/2003" >
        </Project>
        ```

3.  匯入 `MSBuild.Community.Tasks.Targets`

    >*   如果前面是使用 `.msi` 安裝，位置在預設安裝路徑 `C:\Program Files (x86)\MSBuild\MSBuildCommunityTasks` 中
    >*   如果是使用 `.zip` 請直接指向解壓後的目錄位置

    *   簡易寫法

        ```xml
        <Import Project="C:\MSBuild.Community.Tasks.v1.5.0.235\MSBuild.Community.Tasks.Targets"/>
        ```

    *   階層寫法


        *   加入 `PropertyGroup` 並加入 `MSBuild.Community.Tasks.Targets` 路徑

            ```xml
            <PropertyGroup>
                <TPath>C:\MSBuild.Community.Tasks.v1.5.0.235</TPath>
            </PropertyGroup>
            ```


        *   將 `MSBuild.Community.Tasks.Targets` 匯入

            ```xml
            <Import Project="$(TPath)\MSBuild.Community.Tasks.Targets"/>
            ```

        *   兩者效果一樣，只差在如果有其他參數會共用的設定路徑時可以較簡潔

4.  匯入 `參數設定檔`

    > 寫法跟上面一樣有兩種，就不重覆贅述

    ```xml
    <Import Project="c:\properties.xml"/>
    ```

5.  加入 `ItemGroup` 並設定產出檔案及 template 檔案名稱

    *   階層寫法

        ```xml
        <ItemGroup>
            <ConfigFile Include="Web.config">
                <Template>Web.config.template</Template>
            </ConfigFile>
        </ItemGroup>
        ```

    *   平行寫法

        ```xml
        <ItemGroup>
            <ConfigFile Include="Web.config" />
            <Template Include="Web.config.template" />
        </ItemGroup>
        ```

    *   兩者效果相同，第一種寫法結構上看起來有從屬關係，但不同寫法會影響後續參數的使用，如果沒有偏好，建議使用平行寫法

6.  加入執行動作的 Target

    *   6-1. 複製檔案

        > 將 template 複製一份成目標檔案

        ```xml
        <Target Name="CopyFile">
            <Copy
            DestinationFiles="@(ConfigFile)"
            SourceFiles="@(Template)" 
            OverwriteReadOnlyFiles="true"/>
        </Target>
        ```

        *   如果上一步使用的是階層寫法，`SourceFiles` 則需改為 `%(Template)`，關於符號的使用請參考 [Special Characters to Escape](https://msdn.microsoft.com/en-us/library/bb546106.aspx)

    *   6-2. 將 config 中的變數取代為 `參數設定檔` 中的內容

        *   設定名稱為 `Default` 與一開始 `Project` 設定預設執行的 `DefaultTargets` 名稱相同
        *   加上 `DependsOnTargets` 設定，定義前一個必需優先執行的動作
        *   加上 `FileUpdate` element
            *   `Regex` 需使用 regular expression
            *   `Multiline` 是否允許多行
            *   `IgnoreCase` 大小寫是否有差異
            *   `ReplacementText` 指定 `參數設定檔` 的變數設定值

                > 注意變數名稱，因為已透過 `import` 指令匯入，所以需用 `$()`

            *   `Files` 指定取代的目標檔案

        ```xml
        <Target Name="Default" DependsOnTargets="CopyFile">
            <FileUpdate Files="@(ConfigFile)" Regex="\$YowkoTest" ReplacementText="$(YowkoTest)" Multiline="true" IgnoreCase="true" />
        </Target>
        ```

    *  完整內容

        ```xml
        <Project DefaultTargets="Default" xmlns="http://schemas.microsoft.com/developer/msbuild/2003" >
        <PropertyGroup>
            <TPath>C:\MSBuild.Community.Tasks.v1.5.0.235</TPath>
            <PropertyValues>C:\</PropertyValues>
            </PropertyGroup>
        <Import Project="$(TPath)\MSBuild.Community.Tasks.Targets"/>
        <Import Project="$(PropertyValues)\properties.xml"/>
        <!--<Import Project="C:\MSBuild.Community.Tasks.v1.5.0.235\MSBuild.Community.Tasks.Targets"/>
        <Import Project="C:\properties.xml"/>-->
        <ItemGroup>
            <!--<ConfigFile Include="Web.config">
            <Template>Web.config.template</Template>
            </ConfigFile>-->
            <ConfigFile Include="Web.config" />
            <Template Include="Web.config.template" />
        </ItemGroup>
        <Target Name="CopyFile">
            <Copy
            DestinationFiles="@(ConfigFile)"
            SourceFiles="@(Template)" 
            OverwriteReadOnlyFiles="true"/>
        </Target>
        <Target Name="Default" DependsOnTargets="CopyFile">
            <FileUpdate Files="@(ConfigFile)" Regex="\$YowkoTest" ReplacementText="$(YowkoTest)" Multiline="true" IgnoreCase="true" />
        </Target>
        </Project>
        ```

## 實際使用

*   使用 msbuild 指令執行新增的 `.msbuild` 定義檔

    ```
    "C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\amd64\MSBuild.exe" msbuildextension.msbuild
    ```

*   執行結果

    ![4result](https://user-images.githubusercontent.com/3851540/29242249-ff8eb772-7fbb-11e7-9b6e-75c6244c72da.png)

## 心得

比較表

|-|MSBuild Extension Pack|MSBuild Community Tasks|
|--- |--- |--- |
|安裝方式|僅安裝檔 <br/> 包含 binary 但無法直接使用|安裝檔及 zip|
|檔案大小|13.6 MB  <br/>包含 `X86` 、 `X64`、binary|.msi：988KB  
.zip：762KB|
|使用路徑|僅限安裝路徑|安裝路徑 or 使用 ZIP 自訂位置|
|文件|有官網說明網頁|僅有 markdown|


1.  同樣使用安裝檔的情況下，MSBuild Community Tasks 檔案 size 較 MSBuildExtensionPack 小很多

    > MSBuild Community Tasks(988KB) v.s. MSBuildExtensionPack(13.6MB)

2.  同樣使用安裝檔的情況下，MSBuild Community Tasks 無法選擇安裝位置

    > 但有提供 `.zip` 直接使用檔案的方式

3.  MSBuild Community Tasks 使用上較彈性

    > 可以隨意複製 `.zip` 內容至指定位置，即可使用，不需要執行安裝檔，對於沒有權限的環境較友善

4.  MSBuild Community Tasks 文件更少

    > MSBuild Extension Pack 至少有官方說明及 wiki 文件可以參考，MSBuild Community Tasks 就只能翻閱 [TaskDocs.md](https://github.com/loresoft/msbuildtasks/blob/master/Documentation/TaskDocs.md)，有些功能還只能自行嘗試

經過一番測試後，個人傾向使用 MSBuild Community Tasks，使用上功能目前看起差不多，但只要複製檔案就能用實在很吸引人

# 參考資訊

1.  [專案間如何共用 config 設定 - 使用 MSBuildExtensionPack](https://blog.yowko.com/2017/08/shared-config.html)
2.  [MSBuild Community Tasks 下載位置](https://github.com/loresoft/msbuildtasks/releases)
3.  [TaskDocs.md](https://github.com/loresoft/msbuildtasks/blob/master/Documentation/TaskDocs.md)
