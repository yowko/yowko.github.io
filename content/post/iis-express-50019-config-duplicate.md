---
title: "IIS Express 出現 500.19 - 0x800700b7 錯誤？！"
date: 2017-06-28T21:00:00+08:00
lastmod: 2020-09-01T21:00:56+08:00
draft: false
tags: ["Debug","IIS Express","web.config"]
slug: "iis-express-50019-config-duplicate"
aliases:
    - /2017/06/iis-express-50019-config-duplicate.html
---
# IIS Express 出現 500.19 - 0x800700b7 錯誤？！
同事請我協助測試一段程式碼，一如往常的 clone source code，使用 Visual Studio 開啟 .sln，按下 F6 restore NuGet packages and build success，接著 Ctrl+F5 啟動不偵錯，一切都如此熟悉自然，但這前次出現的卻是 `HTTP Error 500.19 - Internal Server Error` 的錯誤而不是預期中的正常執行畫面，詭異的是錯誤訊息只在我的電腦上出現(人品問題又來了？！)

雖說錯誤畫面也不算罕見，但這次看到的錯誤卻不太一樣，解決方式也比較少見，所以特別紀錄一下

## 錯誤訊息

- 訊息內容

    ``` 
    HTTP Error 500.19 - Internal Server Error
    The requested page cannot be accessed because the related configuration data for the page is invalid.
    ```

1.  Detailed Error Information:

    ```
    Module    IIS Web Core
    Notification    Unknown
    Handler    Not yet determined
    Error Code    0x800700b7
    Config Error    There is a duplicate 'oracle.manageddataaccess.client' section defined
    Config File    \\?\D:\Svn_Source\******.Messaging.Web\web.config
    Requested URL    http://localhost:47278/
    Physical Path    
    Logon Method    Not yet determined
    Logon User    Not yet determined
    Request Tracing Directory    C:\Users\yowko.tsai\Documents\IISExpress\TraceLogFiles\
    ```

2.  Config Source:

    ```
    8:     <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
    9:     <section name="oracle.manageddataaccess.client" type="OracleInternal.Common.ODPMSectionHandler, Oracle.ManagedDataAccess" />
    10:     <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    ```

3.  More Information:

    ```
    This error occurs when there is a problem reading the configuration file for the Web server or Web application. In some cases, the event logs may contain more information about what caused this error.
    If you see the text "There is a duplicate 'system.web.extensions/scripting/scriptResourceHandler' section defined", this error is because you are running a .NET Framework 3.5-based application in .NET Framework 4. If you are running WebMatrix, to resolve this problem, go to the Settings node to set the .NET Framework version to ".NET 2". You can also remove the extra sections from the web.config file.
    ```
4.  錯誤截圖

    ![1errormsg](https://user-images.githubusercontent.com/3851540/27622729-84b65c2c-5c0b-11e7-8c6c-32168d18022c.png)

## 開始 Debug

1.  Detailed Error Information 中有提到 Config Error

    > `There is a duplicate 'oracle.manageddataaccess.client' section defined`

    ![2configerror](https://user-images.githubusercontent.com/3851540/27622727-84b374bc-5c0b-11e7-9591-0e6ed0356b3e.png)

2.  檢查 web.config 檔，內容很單純一目瞭然不可能重複，web.config 如下

    ```xml
    <configSections>
        <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
        <section name="oracle.manageddataaccess.client" type="OracleInternal.Common.ODPMSectionHandler, Oracle.ManagedDataAccess" />
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>
    ```

3.  google 到黑大文章 [【茶包射手日記】怪異的web.config HttpHandler重複錯誤](http://blog.darkthread.net/post-2014-04-22-duplicated-handler-entry-in-webconfig.aspx) 可能是吃到上層的 web.config

    *   這是直接啟動 iis express 執行的，看一下專案 property 設定

        ![3webproperty](https://user-images.githubusercontent.com/3851540/27622728-84b3903c-5c0b-11e7-920c-f078b58ff70b.png)

    *   確認沒有上層資料夾，排除吃到上層 web.config 設定的可能

4.  嘗試把 web.config 提示重複的區段註解掉還真的可以正常執行XD

    ```xml
    <configSections>
        <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
        <!--<section name="oracle.manageddataaccess.client" type="OracleInternal.Common.ODPMSectionHandler, Oracle.ManagedDataAccess" />-->
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>
    ```

5.  註解掉後可以正常執行，表示真的有吃到其他 config 內容

    > 以下是兩個可能影響 IIS 的較底層 config

    *   `applicationhost.config`

        > 這是從 IIS7 開始加入的 config 設定，包含所有 site、應用程序、虛擬目錄、application pool 的定義跟 web server global 設定

        *   IIS

            > 在 `system32\inetsrv\config` 資料夾下，紀錄 iis site 相關設定

        *   IIS express
            *   VS 2013 含以前

                > 位於 `%userprofile%\documents\iisexpress\config\` 或是 `%userprofile%\my documents\iisexpress\config\` 資料夾下

            *   VS 2013 開始

                > 位於專案 `.vs\config` 資料夾下

    *   `machine.config`

        > Machine.config 包含全電腦的 dll binding、內建 remoting channels 和 ASP.NET 的 config 設定

        *   32-bit

            > `%windir%\Microsoft.NET\Framework\{version}\config\machine.config`

        *   64-bit

            > `%windir%\Microsoft.NET\Framework64\{version}\config\machine.config`

        *   [version] 有 `v1.0.3705`, `v1.1.4322`, `v2.0.50727` or `v4.0.30319`.

            > v3.0 and v3.5 包含在 v2.0.50727 中. `v4.5.x` and `v4.6.x` 包含在 v4.0.30319 中.

        *   因為 iis express 是 32-bit 所以要找 `%windir%\Microsoft.NET\Framework\{version}\config\machine.config`

6.  最後在 `%windir%\Microsoft.NET\Framework\v4.0.30319\CONFIG\machine.config` 中找到重複設定

    > 註解掉 machine.config 中的區段後，專案就可以正常執行了

## 心得

因為其他同事電腦都沒遇到相同問題，未經證實的猜測：是因為我安裝了 ODAC(Oracle Data Access Component) 相關套件造成的，雖然後來我移除 ODAC 所有套件，這個設定還是沒有被清除，但我還是覺得它的嫌疑最大，當然也可能是我自己找不到確定原因硬要推給它 @@"

剛好透過這個異常錯誤，了解到原來影響 IIS Express web site 的除了 web.config 還有其他 `applicationhost.config` 跟 `machine.config`，再次證實自己沒打好基礎跟自己還有好多不懂的東西！！

# 參考資訊

1.  [【茶包射手日記】怪異的web.config HttpHandler重複錯誤](http://blog.darkthread.net/post-2014-04-22-duplicated-handler-entry-in-webconfig.aspx)
2.  [Where is the IIS Express configuration / metabase file found?](https://stackoverflow.com/questions/12946476/where-is-the-iis-express-configuration-metabase-file-found)
3.  [Where Is Machine.Config?](https://stackoverflow.com/questions/2325473/where-is-machine-config)
4.  [machine.config and processmodel with IIS express](https://stackoverflow.com/questions/4762204/machine-config-and-processmodel-with-iis-express)
5.  [The Configuration System in IIS 7](https://docs.microsoft.com/en-us/iis/get-started/planning-your-iis-architecture/the-configuration-system-in-iis-7?WT.mc_id=DOP-MVP-5002594)
