---
title: "製作 LINQPad 的綠色免安裝可攜版 (Portable)"
date: 2017-06-01T22:30:00+08:00
lastmod: 2021-11-02T22:30:09+08:00
draft: false
tags: ["Tools"]
slug: "linqpad-portable"
aliases:
    - /2017/06/linqpad-portable.html
---

## 製作 LINQPad 的綠色免安裝可攜版 (Portable)

公司的某些環境安全性較高，我沒有安裝軟體的權限需要透過 MIS 協助安裝，雖然只要跟主管說一聲接著發信請 MIS 安裝就好，其實流程單純處理速度也滿快的，但可能是我個性反骨不愛這種處處被限制的感覺，明明我不是要做壞事也是公事需要，所以我儘量在權限允許範圍找其他方法來達到我的目標(乖學生不要學，但嚴格來說我也確實沒有違反公司規章嘛XD)

反正以上不是重點，因為上面的總總原因讓我想來試著做 LINQPad 免安裝版，本來以為要用 [Cameyo](http://www.azofreeware.com/2010/09/cameyo-14.html) 製作但後來看到 LINQPad 官方有介紹作法：請參考 [LINQPad and Portable Deployments](https://www.linqpad.net/PortableDeployment.aspx)，但依官方文件內容實作時發現跟我的實際情境不同，所以紀錄一下

以下將會使用 LINQPad 5 當做 demo 範例 (LINQPad 5 需要 .Net Framwork 4.6 以上，請特別留意)

## 複製必要檔案

1. 要讓 LINQPad 可以正確執行<span style="color:red">只需要下面兩個檔案</span>

    - `LINQPad.exe`
    - `LINQPad.exe.config`

2. 以上兩個檔案請至 LINQPad 安裝資料夾複製即可

    - 提供我環境的位置如下

        ```cmd
        %ProgramFiles(x86)%\LINQPad5\LINQPad.exe
        ```

        ```cmd
        %ProgramFiles(x86)%\LINQPad5\LINQPad.exe.config
        ```

## <span style="color:red">以下設定請依實際需要進行調整，非全部必要</span>

我只用到部份功能，沒用到的功能我會註記，請參考官方文件 [LINQPad and Portable Deployments](https://www.linqpad.net/PortableDeployment.aspx)

## Query files

> 現有的查詢檔案，像我要使用的目標環境沒有對外網路，所以 LINQPad 無法驗證，會使得 LINQPad 喪失 intellisense，所以我就可以把寫好的 query 一併包進去

1. 建立 `queries` 資料夾
2. 複製需要的 query files 進 `queries`

    - 提供我環境的位置如下

        ```cmd
        My Documents/LINQPad Queries
        ```

    - 資料夾位置是可調的

        ![1foldersetting](https://cloud.githubusercontent.com/assets/3851540/26669607/1c10fc84-46e1-11e7-803e-7a70c7922a52.png)

## Extensions and Plugins

> 擴充套件
>
> 因為我想要執行的環境沒有對外網路，所以我得先準備好 `Extensions and Plugins`

- 官方建議(我照做沒作用，但我覺得可能是我誤解他的意思XD)

  - 建立 `plugins` 資料夾
  - 複製需要的 Extensions and Plugins 進 plugins

    - 提供我環境的位置如下

        ```cmd
        C:\Users\yowko.tsai\AppData\Local\LINQPad\NuGet.FW46
        ```

    - 資料夾位置是可以調整的

        ![1foldersetting](https://cloud.githubusercontent.com/assets/3851540/26669607/1c10fc84-46e1-11e7-803e-7a70c7922a52.png)

- 我的做法

  - 在 LINQPad 中按下 `F4` 開啟 References and Properties 設定
  - Export all DLLS to Folder

    ![4export](https://cloud.githubusercontent.com/assets/3851540/26669610/1c34a724-46e1-11e7-9b1d-a11890485406.png)

  - 將匯出的檔案複製至 `plugins`

    - 匯出位置為

        ```cmd
        C:\Users\yowko.tsai\AppData\Local\Temp\LINQPad5\Temp Exports
        ```

    - 只複製所需的 DLL 即可，`Framework Assemblies` 資料夾不需要

        ![5copyexport](https://cloud.githubusercontent.com/assets/3851540/26669611/1c41466e-46e1-11e7-8a8f-97e3d459d661.png)

## Custom Code Snippets

- <span style="color:red">這個我沒有用到</span>
- 官方建議

  - 建立 `snippets` 資料夾
  - 複製需要的 snippets 從 `My Documents/LINQPad Plugins` 進 plugins
  - 資料夾位置是可以調整的

    ![1foldersetting](https://cloud.githubusercontent.com/assets/3851540/26669607/1c10fc84-46e1-11e7-803e-7a70c7922a52.png)

## Custom Data Context Drivers

- <span style="color:red">這個我沒有用到</span>
- 官方建議

  - 建立 `drivers` 資料夾
  - 複製需要的 driver 從 `%LocalAppData%\LINQPad\drivers` 進 drivers

## Connections

> 連線字串

- 將 `ConnectionsV2.xml` 複製至 `LINQPad.exe` 所在位置

  - 提供我環境的位置如下

    ```cmd
    %AppData%\LINQPad\ConnectionsV2.xml
    ```

## Default Namespaces/References for New Queries

> 預設 using 的 namespace 及 dll 參考

- 將 `DefaultQuery.xml` 複製至 `LINQPad.exe` 所在位置

  - 提供我環境的位置如下

    ```cmd
    %AppData%\LINQPad\DefaultQuery.xml
    ```

  - 設定位置 --> 在 LINQPad 中按下 `F4`

    ![3reference](https://cloud.githubusercontent.com/assets/3851540/26669609/1c197bca-46e1-11e7-91e6-585bc493e403.png)

- 官方說 LINQPad 5.06.02 以上版本才支援

## NuGet Source Settings

> 有內部或是自訂 NuGet Server 位置

- <span style="color:red">這個我沒有用到</span>
- 官方建議

  - LINQPad 5.21 以上版本才支援
  - 將 `%AppData%\LINQPad\NuGetSources.xml.` 複製至 `LINQPad.exe` 所在位置
  - 設定來源

    ![2nugetsetting](https://cloud.githubusercontent.com/assets/3851540/26669608/1c175db8-46e1-11e7-886c-aec2dd838df3.png)

## User Preferences

> 個人設定

- 將 `RoamingUserOptions.xml` 複製至 `LINQPad.exe` 所在位置

  - 提供我環境的位置如下

    ```cmd
    %AppData%\LINQPad\RoamingUserOptions.xml
    ```

## 參考資訊

1. [LINQPad and Portable Deployments](https://www.linqpad.net/PortableDeployment.aspx)
