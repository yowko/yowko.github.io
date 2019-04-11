---
title: "關於 Visual Studio 中的外部工具(External Tools)"
date: 2017-06-24T14:27:00+08:00
lastmod: 2018-09-23T15:54:31+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-external-tools"
aliases:
    - /2017/06/visual-studio-external-tools.html
---
# 關於 Visual Studio 中的外部工具(External Tools)
TDD 第三天課程中，有個重點是使用 Pickles 與 SpecFlow 來產生測試及說明文件，做法就是透過 Visual Studio 來執行外部工具，因為之前沒有使用過 Visual Studio 外部工具的 經驗加上練習時發現有些眉眉角角要注意，順手筆記一下

## 預設內建工具

*   Visual Studio 2015 及之前版本
    1.  Create GUID

        > 產生 GUID

    2.  Error Lookup

        > 從輸入的值取得錯誤訊息

    3.  PreEmptive Dotfuscator and Analytics        >保護 .NET 程式免於反向工程
    4.  SPY++ 及 SPY++ x64

        > 以圖形化方式顯示處理序、執行緒、視窗及視窗訊息

    5.  WCF Service Configuration Editor

        > 可用以建立及修改 WCF 服務的組態設定

    ![1vs2015tools](https://user-images.githubusercontent.com/3851540/27506277-c7f492fa-58e7-11e7-8312-10fc1e835a8b.png)

*   Visual Studio 2017
    *   WCF Service Configuration Editor

        > 可用以建立及修改 WCF 服務的組態設定

        ![2vs2017tools](https://user-images.githubusercontent.com/3851540/27506278-c7f9c748-58e7-11e7-952e-ea11fb86b602.png)

    *   其他工具都不再內建，從 vs_installer 中也沒有選項可以裝回來，以 Create GUID 為例，VS 2017 已排除安裝對應的 guidgen.exe

        ![3excludeguid](https://user-images.githubusercontent.com/3851540/27506279-c81e54f0-58e7-11e7-9f12-5ed2ee018d50.png)

## 自訂執行工具

1.  Visual Studio 主選單 Tools --> External Tools...

    ![4addtools](https://user-images.githubusercontent.com/3851540/27506282-c8208d92-58e7-11e7-9ebd-058edf998afa.png)

2.  填寫自訂工具

    ![5addtools2](https://user-images.githubusercontent.com/3851540/27506285-c8498de6-58e7-11e7-945a-1428f8f08acc.png)

    * Title

        > 顯示在選單上的名稱

        ![5title](https://user-images.githubusercontent.com/3851540/27506286-c8495bc8-58e7-11e7-9d37-e0c48f8ed164.png)

    *   Command

        > 執行檔案的位置，支援執行檔格式請參照下一節說明

    *   Arguments

        > 執行時需要參數，選擇的參數說明請參照下方章節

    *   Initial directory

        > 指定執行起始資料夾

    *   Use Output windows

        *   `.exe` 不支援這個選項

            ![5-2exenosupport](https://user-images.githubusercontent.com/3851540/27506281-c8200124-58e7-11e7-8d8f-284d298b905b.png)

        *   與 `Close on exit` 互斥，無法同時使用
        *   將結果輸出於 Visual Studio Output 視窗中

            ![5output](https://user-images.githubusercontent.com/3851540/27506283-c8476cd2-58e7-11e7-9e4e-0ac3587549d4.png)

    * Prompt for arguments

        > 執行時會跳出參數選擇視窗

        ![5-3args](https://user-images.githubusercontent.com/3851540/27506280-c81f3708-58e7-11e7-900c-289695e3226b.png)

    *   Treat output as Unicode

        *   將指令執行結果 output 使用 Unicode 編碼
        *   需搭配 `Use Output windows` 使用
        *   `.exe` 不支援這個選項

            ![5-3exenosupport](https://user-images.githubusercontent.com/3851540/27506289-c8976520-58e7-11e7-8b36-041e38428490.png)

    *  Close on exit

        *   執行結束後關閉
        *   與 `Use Output windows` 互斥，無法同時使用
        *   `.exe` 不支援這個選項

            ![5-4exenosupport](https://user-images.githubusercontent.com/3851540/27506284-c847afc6-58e7-11e7-90d9-b5d5291d097f.png)

## 支援執行格式

1.  .exe
2.  .com
3.  .pif
4.  .bat
5.  .cmd


![6format](https://user-images.githubusercontent.com/3851540/27506287-c870b204-58e7-11e7-99ba-905b25511c93.png)

## 參數


|參數|說明|範例|
|--- |--- |--- |
|$(ItemPath)|目前檔案的完整檔案名稱<br/>(磁碟機 + 路徑 + 檔案名稱)|`C:\Projects\TestForWebApi\TestForWebApi\Views\Home\Index.cshtml`|
|$(ItemDir)|目前檔案的目錄<br/>(磁碟機 + 路徑)|`C:\Projects\TestForWebApi\TestForWebApi\Views\Home\`|
|$(ItemFilename)|目前檔案的檔案名稱 (檔案名稱)|`Index`|
|$(ItemExt)|目前檔案的副檔名|`.cshtml`|
|$(CurLine)|程式碼視窗中滑鼠游標目前的行位置|`10`|
|$(CurCol)|程式碼視窗中滑鼠游標目前的資料行位置|`6`|
|$(CurText)|選取的文字|`<hr/>`|
|$(TargetPath)|要建置之項目的完整檔案名稱<br/>(磁碟機 + 路徑 + 檔案名稱)|`C:\Projects\TestForWebApi\TestForWebApi\obj\Debug\TestForWebApi.dll`|
|$(TargetDir)|要建置之項目的目錄|`"C:\Projects\TestForWebApi\TestForWebApi\obj\Debug\"`|
|$(TargetName)|要建置之項目的檔案名稱|`TestForWebApi`|
|$(TargetExt)|要建置之項目的副檔名|`.dll`|
|$(BinDir)|正在建置之二進位檔的最終位置 (定義為磁碟機 + 路徑)|`C:\Projects\TestForWebApi\TestForWebApi\bin\`|
|$(ProjectDir)|目前專案的目錄 (磁碟機 + 路徑)|`C:\Projects\TestForWebApi\TestForWebApi\`|
|$(ProjectFileName)|目前專案的檔案名稱 (磁碟機 + 路徑 + 檔案名稱)|`TestForWebApi.csproj`|
|$(SolutionDir)|目前方案的目錄 (磁碟機 + 路徑)|`C:\Projects\TestForWebApi\`|
|$(SolutionFileName)|目前方案的檔案名稱 (磁碟機 + 路徑 + 檔案名稱)|`TestForWebApi.sln`|



## 心得

參數不少，光看說明可能不太好理解用途，我的做法是一一將參數輸出來比對，下面提供個人做法

*   方法一：使用指令輸出

    1.  新增一個 .bat 檔
    2.  echo 參數
        *   %1...%9 分別代表第一個參數...直到第九個參數
        *   %0 代表執行檔位置(ex：c:\test.bat)
        *   %* 所有參數

    3.  搭配 `Use Output windows`

        ![7echooutput](https://user-images.githubusercontent.com/3851540/27506288-c87330ba-58e7-11e7-8296-40eff598dbd6.png)

*   方法二：使用 `Prompt for arguments`
    1.  指定隨意執行檔
    2.  勾選 `Prompt for arguments`
    3.  執行自訂工具時選擇參數即會輸出對應指令及參數

    ![8argoutput](https://user-images.githubusercontent.com/3851540/27506276-c7ce19f4-58e7-11e7-822a-fcbd8469f07a.png)

*   另外有個小地方提醒一下：參數之間需要用空白符號隔開，否則將會被當做同個參數，會讓實際行為與預期有落差


# 參考資訊

1.  [管理外部工具](https://msdn.microsoft.com/zh-tw/library/76712d27.aspx)
2.  [Using parameters in batch files at DOS command line](https://stackoverflow.com/questions/14286457/using-parameters-in-batch-files-at-dos-command-line)
