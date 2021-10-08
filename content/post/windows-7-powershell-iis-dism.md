---
title: "Windows 7 中無法使用 PowerShell 安裝 IIS？！改用 DISM"
date: 2018-02-09T01:28:00+08:00
lastmod: 2021-10-08T01:28:46+08:00
draft: false
tags: ["IIS","PowerShell"]
slug: "windows-7-powershell-iis-dism"
aliases:
    - /2018/02/windows-7-powershell-iis-dism.html
---
## Windows 7 中無法使用 PowerShell 安裝 IIS？！改用 DISM

這次遇到的問題是在公司的 Windows 7 電腦上，雖然 Windows 7 也是優秀的作業系統，但終究是較早期的產品，對於一些新的工具就得自行安裝或是不支援，今天遇到的狀況就是其中一個例子

因為安裝 Windows feature 的 GUI 持續吐出錯誤 無法完成安裝，所以想試試透過 PowerShell 安裝，結果 import module 就失敗

發現網路相關文章不多，自己紀錄一下囉

## 無法使用 `ServerManager` module

> * Import-Module ServerManager
> * Add-WindowsFeature

1. 錯誤訊息
    * 訊息內容

        ```txt
            Import-Module : The specified module 'ServerManager' was not loaded because no valid module file was found in any module directory.
            At line:1 char:1
            + Import-Module ServerManager
            + ~~~~~~~~~~~~~~~~~~~~~~~~~~~
                + CategoryInfo          : ResourceUnavailable: (ServerManager:String) [Import-Module], FileNotFoundException
                + FullyQualifiedErrorId : Modules_ModuleNotFound,Microsoft.PowerShell.Commands.ImportModuleCommand
        ```

    * 錯誤截圖

        ![1error](https://user-images.githubusercontent.com/3851540/35987509-d0c1147c-0d36-11e8-94bd-5a1e3ec55b0a.png)

2. 問題發生原因

    > 因為 Windows 7 預設未包含 ServerManager module

## 解決方式：使用 `DISM`

1. 取得所有設定

    ```cmd
    dism /online /get-features /format:table
    ```

    ![2modulelist](https://user-images.githubusercontent.com/3851540/35987870-dde628bc-0d37-11e8-9cfd-fc609add185d.png)

2. 啟用 feature
    * 語法

        ```cmd
        dism /online /enable-feature /featurename:{目標 feature}
        ```

    * 範例

        ```cmd
        dism /online /enable-feature /featurename:IIS-ASPNET  /featurename:IIS-ISAPIExtensions /featurename:IIS-ISAPIFilter /featurename:IIS-NetFxExtensibility
        ```

## 心得

之前我認真覺得 PowerShell 的前景可期，可以透過指令操作 server 執行多數行為，加上不需要 compile 讓使用上非常便利

只是開始使用其他指令工具後，逐漸認為 PowerShell 可能沒有機會贏過其他平台指令，一來是 PowerShell 不僅指令很多還不好記，二來是相關資料沒有統一平台，指令跟範例散落在各處，最後是網頁討論及社群參與太少導致想要用的人找不到適合資源，一來一往就更少人用了

## 參考資訊

1. [Install Internet Information Services (IIS) on Windows 7](https://www.sepago.com/blog/2012/05/19/install-internet-information-services-iis-on-windows-7)
