---
title: "NuGet 發行 Package 時自動增加版號"
date: 2017-07-22T23:56:00+08:00
lastmod: 2021-11-01T17:11:21+08:00
draft: false
tags: ["csharp","Jenkins","NuGet","Visual Studio"]
slug: "nuget-package-auto-version"
aliases:
    - /2017/07/nuget-package-auto-version.html
---
## NuGet 發行 Package 時自動增加版號

之前文章 [Jenkins 自動 Publish NuGet Package](/jenkins-publish-nuget-package) 已經可以透過 Jenkins 將程式碼成品打包成 NuGet package，但文末也提到 NuGet 是使用 {id+版號} 當做唯一值，一般情況下，NuGet server 預設 {id+版號} 相同時就無法上傳 package，不會自動覆蓋，所以今天要來介紹如何讓 NuGet 自動增加版本讓發行時不會出現錯誤

今天分享個人如何使用 DLL 自動增加版號，並使用 DLL 版號來當做 NuGet 版號

## 版本重覆錯誤

- 錯誤訊息

    ``` log
    Failed to process request. 'Package Model.Demo 1.0.0 already exists. The server is configured to not allow overwriting packages that already exist.'.
    ```

- 錯誤截圖

    ![1nugeterror](https://user-images.githubusercontent.com/3851540/28492603-26cf5906-6f39-11e7-9ae0-d02fee128443.png)

## DLL 版本資訊

1. 檢視版本資訊

    > DLL 版本資訊可以在 DLL 檔案的 Details 中找到 File version

    ![2fileversion](https://user-images.githubusercontent.com/3851540/28492604-26f78d04-6f39-11e7-8fb1-26ab10a5507c.png)

2. 如何設定版本

    > DLL 的版本資訊紀錄在專案 `Properties` 資料夾中的 `AssemblyInfo.cs` 內，由 `AssemblyFileVersion` 指定版本，一旦 `AssemblyFileVersion` 不存在則由 `AssemblyVersion` 指定版本

    ```cs
    [assembly: AssemblyVersion("1.0.0.0")]
    [assembly: AssemblyFileVersion("1.0.0.0")]
    ```

3. 版本設定說明

    >版本設定由四個數字組成，分別代表含意如下

    - Major Version
    - Minor Version
    - Build Number
    - Revision

## 讓 DLL 自動增加版號

1. 將 `AssemblyVersion` 使用 `*` 設定版本編號

    > 使用 build 當下時間來自動增加版號

    - 1-1. 僅自動產生 `Revision`

        > `[assembly: AssemblyVersion("1.0.0.*")]`

        - Revision 算法：當地時間午夜至 build 當下時間總秒數差除以 2
        - 兩秒內 build 出的 dll 版號會相同
        - 不同日期可能會出現相同版號

    - 1-2. 自動產生 `Build Number` 與 `Revision`

        > `[assembly: AssemblyVersion("1.0.*")]`

        - Build Number 算法：build 當下時間與 2000/1/1 的天數差

2. 註解掉 `AssemblyFileVersion`
    - dll 版本預設使用 `AssemblyFileVersion`，註解掉後改使用 `AssemblyVersion` 當做版號設定
    - 因為 `AssemblyFileVersion` 不支援自動增加版號

## NuGet Package 使用 DLL 版號做為發行版號

以下設定延續 [Jenkins 自動 Publish NuGet Package](/jenkins-publish-nuget-package) 內容，需修改 `NuGetPackage.ps1`

1. 取得 dll 的版號

    - 在 277 行加上下列程式碼 (如果行號跟程式對不起來，請搜尋 `Creating package...`)

        ```ps1
        $dllversion = get-childitem .\lib -recurse -include *.dll| foreach-object { "{0}" -f [System.Diagnostics.FileVersionInfo]::GetVersionInfo($_).FileVersion }|Select-Object -First 1
        ```

    - 取出第一個 `lib` 資料夾下的 dll 檔，再讀取 dll 檔的版號

        > dll 會由前一個建置動作產生至 `lib`，接著使用 PowerShell 取得 dll 檔案版本資訊

2. 發行指令加上版號參數

    - 280 行加上指定版號(如果行號跟程式對不起來，請搜尋 `pack`)
        - 原程式碼

            ```ps1
            $packageTask = Create-Process .\NuGet.exe ("pack Package.nuspec -Symbol -Verbosity Detailed")
            ```

        - 修改後程式

            ```ps1
            $packageTask = Create-Process .\NuGet.exe ("pack Package.nuspec -Version "+ "$dllversion" +" -Symbol -Verbosity Detailed")
            ```

    - 292 行加上指定版號(如果行號跟程式對不起來，請搜尋 `pack`)
        - 原程式碼

            ```ps1
            $packageTask = Create-Process .\NuGet.exe ("pack Package.nuspec -Verbosity Detailed")
            ```

        - 修改後程式碼

            ```ps1
            $packageTask = Create-Process .\NuGet.exe ("pack Package.nuspec -Version "+ "$dllversion" +" -Verbosity Detailed")
            ```

## 修改成果

1. Jenkins 成功建置並順利發行 NuGet package

    ![3jenkins](https://user-images.githubusercontent.com/3851540/28492605-2716ccd2-6f39-11e7-9250-790faed14f6c.png)

2. NuGet 可以找到正確版本

    ![4newversion](https://user-images.githubusercontent.com/3851540/28492606-271d49e0-6f39-11e7-93e5-5d062d599ca2.png)

## 心得

透過建置產生自動設定版號的確是方便很多，但這個版號對 user 來說並不友善，只能算是個便宜行事的解決方案，難登大雅之堂，但可以解決之前 {id + 版本} 需要唯一的問題，待專案壓力稍緩或是 user 報怨時再來調整

## 參考資訊

1. [AssemblyVersionAttribute Class](https://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute.aspx)
2. [Automatically generating version numbers in Visual Studio](https://jonthysell.com/2017/01/10/automatically-generating-version-numbers-in-visual-studio/)
3. [Jenkins 自動 Publish NuGet Package](/jenkins-publish-nuget-package)
