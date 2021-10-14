---
title: "Visual Studio 無法使用 Yarn 安裝套件"
date: 2017-10-27T00:42:34+08:00
lastmod: 2021-10-13T00:42:34+08:00
draft: false
tags: ["Visual Studio","Tools"]
slug: "visual-studio-yarn"
aliases:
    - /2017/10/visual-studio-yarn.html
---
## Visual Studio 無法使用 Yarn 安裝套件

最近新專案開發，不像以前有專責的前端工程師協助前端頁面處理，一切都要自己來，所以就出現需要在 Visual Studio 安裝前端套件的現象，今天遇到的問題是透過 Visual Studio 2015 使用 yarn 要安裝 vue.js 套件無法正確執行，就來看看該如何解決吧

## 操作步驟

1. 專案按右鍵 --> Quick Install Package...

    ![0install1](https://user-images.githubusercontent.com/3851540/32112595-42b0d59a-bb70-11e7-8802-69104964fda0.png)

2. 選擇 `Yarn` --> 輸入套件名稱

    ![0install2](https://user-images.githubusercontent.com/3851540/32112596-42d7954a-bb70-11e7-9191-f6c84f76d891.png)

## 錯誤訊息

- 訊息內容

    ```txt
    Installing pure-vue-select package from Yarn...
    'yarn' is not recognized as an internal or external command,operable program or batch file.
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/32112597-42fd72d8-bb70-11e7-8e68-7c77077dd957.png)

## 問題確認

1. 開啟 Package Manager Console

    ![2pmc](https://user-images.githubusercontent.com/3851540/32112598-4323403a-bb70-11e7-906f-3769288b8b9f.png)
2. 確認 Yarn 指令是否可以正確使用
    - 執行 `yarn --version`
    - 出現錯誤訊息
        - 訊息內容

        ```txt
        The term 'yarn' is not recognized as the name of a cmdlet, function, script file, or operable  program. Check the spelling of the name, or if a path was included, verify that the path is correct  and try again.At line:1 char:1
        yarn --version
        
        + yarn --version
        + ~~~~
            + CategoryInfo          : ObjectNotFound: (yarn:String) [], CommandNotFoundException
            + FullyQualifiedErrorId : CommandNotFoundException
        ```

        - 錯誤截圖

            ![3yarnerror](https://user-images.githubusercontent.com/3851540/32112599-43547592-bb70-11e7-8085-b9f704214dd6.png)
3. 開啟 command prompt 執行 `yarn --version`

    ![4cmdyarn](https://user-images.githubusercontent.com/3851540/32112600-437a29e0-bb70-11e7-809b-340d2ef698e3.png)

## 解決方式

看到上述錯誤後可以確認問題是 Visual Studio 找不到 `yarn.cmd` 檔案位置造成的，但在 command prompt 中指令又可以正確執行，這是因為 command prompt 使用的環境變數包含 `System Variables` 與 `User Variables` 的 `Path` 的內容，而 Visual Studio 則是使用環境變數中 `System Variables` 的 `Path` 內容

1. 將 Yarn.cmd 所在資料夾加入環境變數中 `System Variables` 的 `Path`

    >使用 PowerShell 執行加入指令

    - 格式
        > `{username}` 請換成正確使用者名稱

        ```ps1
        $AddedLocation =";C:\Users\{username}\AppData\Roaming\npm"
        $Reg = "Registry::HKLM\System\CurrentControlSet\Control\Session Manager\Environment"
        $OldPath = (Get-ItemProperty -Path "$Reg" -Name PATH).Path
        $NewPath= $OldPath + $AddedLocation
        Set-ItemProperty -Path "$Reg" -Name PATH –Value $NewPath
        ```

    - 實例

        ```ps1
        $AddedLocation =";C:\Users\yowko.tsai\AppData\Roaming\npm"
        $Reg = "Registry::HKLM\System\CurrentControlSet\Control\Session Manager\Environment"
        $OldPath = (Get-ItemProperty -Path "$Reg" -Name PATH).Path
        $NewPath= $OldPath + $AddedLocation
        Set-ItemProperty -Path "$Reg" -Name PATH –Value $NewPath
        ```

2. 重啟 Visual Studio

    > 重啟 Visual Studio 才能吃到新的環境變數設定

3. 正常使用

    ![5yarnok](https://user-images.githubusercontent.com/3851540/32112601-43a04242-bb70-11e7-9ff6-88a2f4be56e0.png)

    ![6yarninstalled](https://user-images.githubusercontent.com/3851540/32112603-43c65720-bb70-11e7-88d0-0a7c1c8e6e4b.png)

## 心得

問題解決方式很普遍，發生原因也很容易猜想的到，主要是想紀錄一下一般 command prompt 使用環境變數與 Visual Studio 使用環境變數的不同，另外就是使用 PowerShell 修改環境變數的語法

## 參考資訊

1. [PowerShell's Path Environmental Variable](http://www.computerperformance.co.uk/powershell/powershell_env_path.htm)
