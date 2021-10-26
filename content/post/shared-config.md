---
title: "專案間如何共用 config 設定 - 使用 MSBuildExtensionPack"
date: 2017-08-13T00:24:00+08:00
lastmod: 2021-10-26T00:24:22+08:00
draft: false
tags: ["套件","MSBuild","web.config"]
slug: "shared-config"
aliases:
    - /2017/08/shared-config.html
---
## 專案間如何共用 config 設定 - 使用 MSBuildExtensionPack

以往專案公司的經驗中，較常見情境是一個專案中可能會針對不同的環境建立不同的組態設定( configuration )，在發行 (publish) 時透過 configuration transformation 來將 config 設定為相對應的內容

而以產品為主的公司，需求就有些不同，相同的是還是需要為不同環境設定不同的 config，但卻多了不同專案間共用 config 設定的需求，以 `連線字串` 為例，在多個專案間都需要相同連線字串，而這個連線字串如果需要更新時，就是所有使用到這個連線字串的專案都要逐一調整，否則就會出現某幾個專案漏改的情況

面對上述情境，configuration transformation 就無法派上用場，這時就可以透過 msbuild 的相關功能來處理

## 安裝 MSBuildExtensionPack

MSBuildExtensionPack 是一個 MSBuild 的擴充套件，大大地強化了 MSBuild 的功能，今天的需求就是要借助 MSBuildExtensionPack 的幫忙，關於 MSBuildExtensionPack 的詳細內容請參考 [MSBuildExtensionPack](https://github.com/mikefourie/MSBuildExtensionPack)

* [下載位置](https://github.com/mikefourie/MSBuildExtensionPack/releases)

* 解壓縮後依 OS 版本來執行 `.msi`

    ![1install](https://user-images.githubusercontent.com/3851540/29242247-ff8440bc-7fbb-11e7-95e5-160d30297e04.png)

* 記得保留安裝位置

    > 後續設定會用到

    ![2installpath](https://user-images.githubusercontent.com/3851540/29242248-ff88e568-7fbb-11e7-97cf-2aebed52426f.png)

* 安裝完成

    ![3installed](https://user-images.githubusercontent.com/3851540/29242250-ff909fce-7fbb-11e7-9464-fbec1e9482f4.png)

## 在專案資料夾中新增 config template

1. 複製一份 config 並 rename 成其他名稱
    * 範例：

        > `Web.config.template`

2. 將 config 內容想要由共用設定決定的部份改成變數
    * 範例：
        * 在 appSettings 區段中加上一個 `TestGlobalSetting`
        * value 則使用變數 `$YowkoTest`

    * 完整內容

        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <configuration>
            <appSettings>
                <add key="TestGlobalSetting" value="$YowkoTest" />
            </appSettings>
        </configuration>
        ```

## 在共用資料夾中新增共用的參數設定檔

* 檔案不需要跟特定專案放在一起，方便與其他專案共用
* 關於 element 的詳細說明可以參考 [MSBuild Properties](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-properties?WT.mc_id=DOP-MVP-5002594)

1. 使用 xml 格式

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    ```

2. 以 `Project` 為 root element

    ```xml
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    </Project>
    ```

3. 加入 `PropertyGroup` element

    ```xml
    <PropertyGroup>
    </PropertyGroup>
    ```

4. 將目標參數值以參數 element 包覆

    ```xml
    <YowkoTest>ValueChanged</YowkoTest>
    ```

5. 完整內容

    ```xml
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <PropertyGroup>
            <YowkoTest>ValueChanged</YowkoTest>
        </PropertyGroup> 
    </Project>
    ```

6. 範例檔名：`properties.xml`；路徑：`C:\properties.xml`

## 在專案資料夾中新增 msbuild 定義檔

* 用來執行 config 內容取代作業

1. 使用 xml 格式

2. 以 `Project` 為 root element 並指定 `ToolsVersion` 與 `DefaultTargets`

    * ToolsVersion

        > 只有 `3.5` 與 `4.0` 兩個選擇，3.5 支援 .NET Framework 3.5；4.0 則支援 .NET Framework 4.0 以上

    * DefaultTargets

        > 宣告預設執行的 `Target` 內容，稍後會定義，名稱可自訂

    * 範例：

        ```xml
        <Project ToolsVersion="4.0" DefaultTargets="Default" xmlns="http://schemas.microsoft.com/developer/msbuild/2003" >
        </Project>
        ```

3. 將 `MSBuild.ExtensionPack.tasks` 與 `參數設定檔` 匯入

    > `MSBuild.ExtensionPack.tasks`存於 MSBuildExtensionPack 安裝目錄下

    * 簡易寫法

        ```xml
        <Import Project="C:\Program Files\MSBuildExtensionPack\4.0\MSBuild.ExtensionPack.tasks"/>
        <Import Project="C:\properties.xml"/>
        ```

    * 結構寫法

        * 加入 `PropertyGroup` 並加入 `MSBuild.ExtensionPack.tasks` 與 參數設定檔 路徑

            ```xml
            <PropertyGroup>
                <TPath>C:\Program Files\MSBuildExtensionPack\4.0</TPath>
                <PropertyValues>C:\</PropertyValues>
            </PropertyGroup>
            ```

        * 將 `MSBuild.ExtensionPack.tasks` 與 `參數設定檔` 匯入

            ```xml
            <Import Project="$(TPath)\MSBuild.ExtensionPack.tasks"/>
            <Import Project="$(PropertyValues)\properties.xml"/>
            ```

    * 兩者效果一樣，只差在如果有其他參數會共用的設定路徑時可以較簡潔

4. 加入 `ItemGroup` 並設定產出檔案及 template 檔案名稱

    * 階層寫法

        ```xml
        <ItemGroup>
            <ConfigFile Include="Web.config">
                <Template>Web.config.template</Template>
            </ConfigFile>
        </ItemGroup>
        ```

    * 平行寫法

        ```xml
        <ItemGroup>
            <ConfigFile Include="Web.config" />
            <Template Include="Web.config.template" />
        </ItemGroup>
        ```

    * 兩者效果相同，第一種寫法結構上看起來有從屬關係，但不同寫法會影響後續參數的使用，如果沒有偏好，建議使用平行寫法

5. 加入執行動作的 Target

    * 5-1. 複製檔案

        > 將 template 複製一份成目標檔案

        ```xml
        <Target Name="CopyFile">
            <Copy
            DestinationFiles="@(ConfigFile)"
            SourceFiles="@(Template)" 
            OverwriteReadOnlyFiles="true"/>
        </Target>
        ```

        * 如果上一步使用的是階層寫法，`SourceFiles` 則需改為 `%(Template)`，關於符號的使用請參考 [Special Characters to Escape](https://msdn.microsoft.com/en-us/library/bb546106.aspx)

    * 5-2. 將 config 中的變數取代為 `參數設定檔` 中的內容

        * 設定名稱為 `Default` 與一開始 `Project` 設定預設執行的 `DefaultTargets` 名稱相同
        * 加上 `DependsOnTargets` 設定，定義前一個必需優先執行的動作
        * 加上 `MSBuild.ExtensionPack.FileSystem.File` element
            * `TaskAction` 使用 `Replace`
            * `RegexPattern` 需使用 regular expression
            * `RegexOptionList` 依需求指定
            * `Replacement` 指定 `參數設定檔` 的變數設定值

                > 注意變數名稱，因為已透過 `import` 指令匯入，所以需用 `$()`

            * `Files` 指定取代的目標檔案

        ```xml
        <Target Name="Default" DependsOnTargets="CopyFile">
            <MSBuild.ExtensionPack.FileSystem.File TaskAction="- " RegexPattern="\$YowkoTest" RegexOptionList="IgnoreCase|Singleline" Replacement="$(YowkoTest)" Files="@(ConfigFile)"/>
        </Target>
        ```

    * 完整內容

        ```xml
        <Project ToolsVersion="4.0" DefaultTargets="Default" xmlns="http://schemas.microsoft.com/developer/msbuild/2003" >
        <PropertyGroup>
            <TPath>C:\Program Files\MSBuildExtensionPack\4.0</TPath>
            <PropertyValues>C:\</PropertyValues>
        </PropertyGroup>
        <Import Project="$(TPath)\MSBuild.ExtensionPack.tasks"/>
        <Import Project="$(PropertyValues)\properties.xml"/>
        <!--<Import Project="C:\Program Files\MSBuildExtensionPack\4.0\MSBuild.ExtensionPack.tasks"/>
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
            <MSBuild.ExtensionPack.FileSystem.File TaskAction="Replace" RegexPattern="\$YowkoTest" RegexOptionList="IgnoreCase|Singleline" Replacement="$(YowkoTest)" Files="@(ConfigFile)"/>
        </Target>
        </Project>
        ```

## 實際使用

* 使用 msbuild 指令執行新增的 `.msbuild` 定義檔

    ```cmd
    "C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\amd64\MSBuild.exe" msbuildextension.msbuild
    ```

* 執行結果

    ![4result](https://user-images.githubusercontent.com/3851540/29242249-ff8eb772-7fbb-11e7-9b6e-75c6244c72da.png)

## 心得

文章內容不多，設定也不難，但花了不少時間看文件，尤其 msbuild 的內容很多，查了很多資料才搞清楚 msbuild 的 element 及 attribute 的定義及用法，加上 MSBuildExtensionPack 的範例資料也很少，試了好久才搞定

我覺得這個需求應該很普遍，可能有更簡單的做法，應該是我關鍵字下錯才會兜一大圈，但多學一種方法也不錯，以備不時之需

## 參考資訊

1. [MSBuildExtensionPack](https://github.com/mikefourie/MSBuildExtensionPack)
2. [MSBuild Properties](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-properties?WT.mc_id=DOP-MVP-5002594)
3. [Special Characters to Escape](https://msdn.microsoft.com/en-us/library/bb546106.aspx)
