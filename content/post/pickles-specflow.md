---
title: "使用 Pickles 搭配 SpecFlow 產生即時更新文件(living documentation)"
date: 2017-06-27T22:47:00+08:00
lastmod: 2021-10-29T22:47:48+08:00
draft: false
tags: ["MSTest","PowerShell","Tools","Unit Test"]
slug: "pickles-specflow"
aliases:
    - /2017/06/pickles-specflow.html
---
## 使用 Pickles 搭配 SpecFlow 產生即時更新文件(living documentation)

你有遇過類似的狀況嗎？：改了 code 卻忘了改註解 或是 看既有 code 時發現 code 行為與註解說明不同？ 這兩種情況在我的工作經驗中不算是常見但也不算是罕見，其實我倒不認為造成問題的人(可能就是你我)是故意的，大多數都是不小心忘記或是漏了，畢竟人難免會犯錯，功能正常是第一優先也比較容易確認，註解的正確性相對難以保障。所以就會造成文件日益失去可靠性，程式也就愈來愈難維護

當然程式可以自我解釋是最好，但並不是所有人都看得懂程式，老闆問商業邏輯時總不能叫他自己去看程式碼，而身為工程師的大家，我想應該都不喜歡寫文件。加上會隨著需求異動修改的也通常只有程式碼，所以如果可以從程式碼來產生文件就是最理想的狀態了。

一般情境下從程式碼來產生文件當然是很難達成，但如果專案中有使用 SpecFlow 時，事情就變得容易了，今天就來看看如何透過 Pickles 與 SpecFlow 搭配產生即時更新文件

## What is Pickles

Pickles 是套 open source 的即時更新文件(living documentation)產生工具，可以針對 Gherkin 語法(ex. Cucumber、SpecFlow)寫出的 feature 檔來產生文件

## 安裝 Pickles

Pickles 使用上有兩個類型：

1. Pickles

    >* 提供 Pickles 核心功能
    >* 需搭配 runner 使用

    * 安裝方式

        * 使用 Chocolatey 指令

            ```cmd
            choco install pickles
            ```

            > 詳細內容可以參考 [Pickles - The Open Source Living](https://chocolatey.org/packages/pickles)

        * 直接下載 .zip

            > 至 [GitHub](https://github.com/picklesdoc/pickles/releases) 下載

    * 安裝 runner

        * 使用 NuGet 安裝

            * PowerShell

                ```ps1
                Install-Package Pickles
                ```

                > 詳細內容可以參考 [Pickles](https://www.nuget.org/packages/Pickles/)

            * Command Line

                ```ps1
                Install-Package Pickles.CommandLine
                ```

                > 詳細內容可以參考 [Command line version of Pickles](https://www.nuget.org/packages/Pickles.CommandLine/)

            * MSBuild

                ```ps1
                Install-Package Pickles.MSBuild
                ```

                > 詳細內容可以參考 [MSBuild Task for Pickles](https://www.nuget.org/packages/Pickles.MSBuild/)

        * 直接下載 .zip
            * Pickles-powershell.zip
            * Pickles-msbuild.zip

            > 至 [GitHub](https://github.com/picklesdoc/pickles/releases) 下載

2. Pickles UI

    > * 以 GUI 方式來執行 Pickles
    > * 內建 runner 不需額外安裝

    * 安裝方式

        * 使用 Chocolatey 指令

            ```ps1
            choco install picklesui
            ```

            > 詳細內容可以參考 [Pickles UI - The Open Source Living](https://chocolatey.org/packages/picklesui)

        * 直接下載 .zip

            > 至 [GitHub](https://github.com/picklesdoc/pickles/releases) 下載

## [參數](http://docs.picklesdoc.com/en/latest/Arguments/)

* Feature Directory

    > Feature 檔位置資料夾

  * Console

    ```cmd
    Pickles.exe --feature-directory=C:\MyProject\Features
    Pickles.exe -f=C:\MyProject\Features
    ```

  * Powershell

    ```ps1
    Pickle-Features -FeatureDirectory C:\MyProject\Features
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles FeatureDirectory="C:\MyProject\Features" />
    </Target>
    ```

* Output Directory

    > 報表產出目錄

  * Console

    ```cmd
    Pickles.exe --output-directory=C:\GeneratedDocs
    Pickles.exe -o=C:\GeneratedDocs
    ```

  * Powershell

    ```ps1
    Pickle-Features -OutputDirectory C:\GeneratedDocs
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles OutputDirectory="C:\GeneratedDocs" />
    </Target>
    ```

* Documentation Format

    > 指定輸出文件的格式 預設為 `html` 可以使用 html, dhtml, excel, json, word, cucumber

  * Console

    ```cmd
    Pickles.exe --documentation-format=Word
    Pickles.exe -df=Word
    ```

  * Powershell

    ```ps1
    Pickle-Features -DocumentationFormat Word
    ```

  * MSBuild

    ```xml
    <target Name="document">
        <pickles DocumentationFormat="Word" />
    </target>
    ```

  * html 報表

    ![1html](https://user-images.githubusercontent.com/3851540/27592692-a2023604-5b87-11e7-9d4e-e4be699d81d5.png)

  * dhtml 報表

    ![2dhtml](https://user-images.githubusercontent.com/3851540/27592694-a22b70f0-5b87-11e7-9889-a232e926f214.png)

* System Under Test Name

    > 測試系統名稱，因為 Gherkin 是跨平台的，無法統一從 metadata 讀取系統名稱，所以需要由使用者自行指定(我沒用過)

  * Console

    ```cmd
    Pickles.exe --system-under-test-name=MyProject
    Pickles.exe -sn=MyProject
    ```

  * Powershell

    ```ps1
    Pickle-Features -SystemUnderTestName MyProject
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles SystemUnderTestName="MyProject" />
    </Target>
    ```

* System Under Test Version

    > 測試系統版本，原因同上

  * Console

    ```cmd
    Pickles.exe --system-under-test-version=2.0.1beta
    Pickles.exe -sv=2.0.1beta
    ```

  * Powershell

    ```ps1
    Pickle-Features -SystemUnderTestVersion 2.0.1beta
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles SystemUnderTestVersion="2.0.1beta" />
    </Target>
    ```

* Test Results Format

    > 測試結果的 test framework 格式，預設為 `nunit` 可接受 nunit, nunit3, xunit, xunit2, mstest, cucumberjson, specrun, vstest

  * Console

    ```cmd
    Pickles.exe --test-results-format=mstest
    Pickles.exe -trfmt=mstest
    ```

  * Powershell

    ```ps1
    Pickle-Features -TestResultsFormat mstest
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles ResultsFormat="mstest" />
    </Target>
    ```

* Test Results File

    > 指定 xml(trx) 格式的測試結果檔案

  * Console

    ```cmd
    Pickles.exe --link-results-file=C:\MyProject\Reports\results.trx
    Pickles.exe -lr=C:\MyProject\Reports\results.trx
    Pickles.exe -lr=C:\MyProject\Reports\*.trx
    Pickles.exe -lr=C:\MyProject\UnitTest\results.xml;C:\MyProject\SystemTest\*
    ```

  * Powershell

    ```ps1
    Pickle-Features -TestResultsFile C:\MyProject\Reports\results.trx
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles ResultsFile="C:\MyProject\Reports\results.trx" />
    </Target>
    ```

  * 有指定測試結果  報告上會顯示測試結果

    ![3testresult](https://user-images.githubusercontent.com/3851540/27592699-a29e8036-5b87-11e7-8b37-3aa51ea08401.png)

* Language

    > feature 檔的語言 預設是 `en` (English)，不支援 `zh`

  * Console

    ```cmd
    Pickles.exe --language=sv
    Pickles.exe -l=sv
    ```

  * Powershell

    ```ps1
    Pickle-Features -Language sv
    ```

  * MSBuild

    ```xml
    <Target Name="document">
        <Pickles Language="sv" />
    </Target>
    ```

* Include Experimental Features

    > 是否啟用實驗性功能(我沒用過)

  * Console

    ```cmd
    Pickles.exe --include-experimental-features
    Pickles.exe -exp
    ```

  * Powershell

    ```ps1
    Pickle-Features -IncludeExperimentalFeatures
    ```

  * MSBuild

    ```xml
    <target Name="document">
        <pickles IncludeExperimentalFeatures="true" />
    </target>
    ```

## 如何使用 Pickles

Console (Pickles.exe) 是最容易也是用途最廣跟最普遍的方式；Pickles Powershell(Pickle-Features) 只能在 Visual Studio 中使用；MSBuild 每次 build project 都產一份報表？！

以下 demo 會使用 Console 也就是透過 Pickles.exe 來產生報表搭配 Visual Studio 外部工具，過程會使用 PowerShell 語法(這邊用的 PowerShell 不是上面說的 Pickles Powershell：Pickle-Features)

1. 建立一個 PowerShell 檔

    * 用來執行測試、產生報表、開啟報表
    * `MSTest.exe 的檔案位置` 及 `Pickles-exe 的檔案位置` 需視實際情境修改

        ```ps1
        # 用來接收傳入的參數
        Param
        (
            [Parameter(Mandatory = $true)]
            [String]$TargetName ,
            [String]$TargetExt ,
            [String]$ResultPath ,
            [String]$FeaturePath
        )
        # MSTest.exe 的檔案位置
        $MSTestPath = "C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\MSTest.exe"
        # Pickles-exe 的檔案位置
        $PicklesPath ="D:\Downloads\Pickles-exe-2.16.0\Pickles.exe"
        # MSTest 測試 function; $TestDllPath 是 測試專案 dll 檔案位置(完整檔名); $ResultPath 是測試結果產生位置(完整檔名)
        function ExeTest($MSTestPath,$TestDllPath,$TestResultFilePath)
        {
            & $MSTestPath /testcontainer:$TestDllPath /resultsfile:$TestResultFilePath
        }
        # 產生 Pickles 報表 function; $FeaturePath 是 feature 檔目錄位置; $ResultPath 是報告產出位置 ; $TestResultFilePath 是測試結果檔案位置(完整檔名)
        function GenerateReport($PicklesPath,$FeaturePath,$ResultPath,$ResultFilePath)
        {
            &$PicklesPath --test-results-format=mstest --feature-directory="$FeaturePath " --output-directory="$ResultPath " --link-results-file="$TestResultFilePath " --documentation-format=Dhtml
        }
        # 開啟報表 function ;呼叫 chrome 開啟 Pickles 報表;$ResultIndexPath 是 Pickles 報表位置
        function OpenReport($ResultIndexPath)
        {
            Start-Process "chrome.exe" $ResultIndexPath
        }
        # 使用專案名稱做為測試結果儲存檔名，並指定儲存位置
        $TestResultFilePath=$ResultPath+'\'+$TargetName+'.trx'
        # 如果檔案已存在則刪除(不刪，新的測試結果不會自行覆蓋)
        if( Test-Path -Path $TestResultFilePath) 
        {
            Remove-Item $TestResultFilePath
        }
        # 依各 function 依序執行 測試、產生 pickles 報表、開啟報表
        ExeTest -MSTestPath $MSTestPath -TestDllPath $TargetName$TargetExt -TestResultFilePath $TestResultFilePath | GenerateReport -PicklesPath $PicklesPath -FeaturePath $FeaturePath -ResultPath $ResultPath -TestResultFilePath $TestResultFilePath | OpenReport -ResultIndexPath $ResultPath'/index.html'
        ```

2. 設定 Visual Studio 外部工具

    > 在 [關於 Visual Studio 中的外部工具(External Tools)](/2017/06/visual-studio-external-tools.html) 中有簡單介紹 Visual Studio 外部工具相關使用方式及參數

    * Visual Studio 主選單 Tools --> External Tools

        ![4addexternal](https://user-images.githubusercontent.com/3851540/27592695-a243b43a-5b87-11e7-989c-ee282bdd2562.png)

    * 填寫外部工具資訊
        * Title
            * 用來顯示在 Tools 選單上的名稱

                ![5title](https://user-images.githubusercontent.com/3851540/27592697-a24ae638-5b87-11e7-930f-15237408f829.png)

        * Command

            * 執行檔完整檔名
            * 範例：`Powershell.exe`

        * Arguments

            * 執行檔所需參數，需注意欄位有長度限制
            * 執行 `GeneratePicklesDoc.ps1` (上面新增的 powershell 檔案)，並依參數名稱指定參數值

                * `-executionpolicy remotesigned -File D:\GeneratePicklesDoc.ps1` 執行特定 powershell 檔案
                * `-TargetName $(TargetName)` 指定 `TargetName` 為 build 出來的 dll 檔名
                * `-TargetExt $(TargetExt)` 指定 `TargetExt` 為 build 出來的檔案副檔名
                * `-ResultPath "D:\20170626"` 指定 `ResultPath` 為報告產出位置(測試報告及 pickles 文件會放一起，如需修改請改上面的 ps 檔)
                * `-FeaturePath $(ProjectDir)` 指定 `FeaturePath` 為測試專案的 feature 檔實際路徑(需視專案位置調整)

            * 範例：

                ```ps1
                -executionpolicy remotesigned -File  D:\GeneratePicklesDoc.ps1 -TargetName $(TargetName) -TargetExt $(TargetExt) -ResultPath "D:\20170626" -FeaturePath $(ProjectDir)
                ```

        * Initial directory

            * 執行檔發動根目錄位置，可以縮短參數值長度(不用給到完整路徑)
            * 範例：`$(BinDir)`

3. 產生結果

    * 開啟測試專案後，直接按下剛剛新增的 Visual Studio 外部工具即可自動進行測試、產生文件並開啟文件了

    ![6result](https://user-images.githubusercontent.com/3851540/27592696-a24b55f0-5b87-11e7-8bd6-b78d1c083928.png)

## 心得

Pickles 的說明文件 [Pickles Docs](http://docs.picklesdoc.com/en/latest/) 對新進的使用者並不友善，我反覆看著文件數次，還是搞不太清楚要裝哪些東西跟安裝套件的用途

設定 Pickles 過程中遇到還遇到不熟悉 MSTest.exe 及 Visual Studio 外部工具用法，也發現 MSTest.exe 尚不支援 MSTestV2 跟 Visual Studio 外部工具 參數列有長度限制，整個設定過程上有比較多眉眉角角的，使用上要特別留意，不過將整個流程串接起來讓開發不再只是開發，相信對日後維護有一定的幫助

## 參考資訊

1. [Pickles - GitHub](https://github.com/picklesdoc/pickles)
2. [Pickles Docs](http://docs.picklesdoc.com/en/latest/)
3. [Pickles Arguments](http://docs.picklesdoc.com/en/latest/Arguments/)
