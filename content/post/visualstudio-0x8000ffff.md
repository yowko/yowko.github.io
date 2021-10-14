---
title: "Visual Studio 2017 啟動 Azure Function 偵錯出現 Fatal Error ？！"
date: 2018-11-22T23:45:00+08:00
lastmod: 2021-10-13T23:44:30+08:00
draft: false
tags: ["Visual Studio 2017","Azure Function","dotnet core"]
slug: "visualstudio-0x8000ffff"
---
## Visual Studio 2017 啟動 Azure Function 偵錯出現 Fatal Error ？！

因緣際會下，剛好有個很趕但很小的功能可以透過 Azure Function 來完成，過去 Azure Function 1.x 我來不及參與，直接使用 Azure Function 2.x 似乎也沒什麼違和感，Azure Function 1.x 與 Azure Function 2.x 的相關比較改天有機會再來紀錄，今天先來紀錄一下靈異狀況：透過 Visual Studio 2017 建立 Azure Function 後也快速地完成需要的功能，正要進行 debug 時卻出現過去從沒見過的 fatal error，雖然最後還是不清楚問題發生原因及真正的解決方式，但留個過程備查

## 環境說明

1. Window 10 - 1803 (OS Build 17134.407)

    ![2winver](https://user-images.githubusercontent.com/3851540/48925448-9e50cf00-eefe-11e8-8a74-0b30587aa2f2.png)

2. Visual Studio 2017 - 15.8.4

    ![3vsver](https://user-images.githubusercontent.com/3851540/48925449-9e50cf00-eefe-11e8-8f03-384491c9521b.png)

3. Azure Function v2 Preview (.NET Standard)

    ![4afpreview](https://user-images.githubusercontent.com/3851540/48925450-9e50cf00-eefe-11e8-830b-4f9e75ffc8e3.png)

## 錯誤訊息

1. 訊息內容

    ```txt
    A fatal error has occurred and debugging needs to be terminated. For more details, please se the Microsoft Help and Support web site. HRESULT = 0x8000ffff. ErrorCode = 0x0.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/48925447-9e50cf00-eefe-11e8-8d37-561cee567cb9.png)

## 解決過程與方式

1. 解決方案侯選人
    - MSDN 有相同錯誤代碼問題 [catastrophic failure (exception from hresult 0x8000ffff (e_unexpected)) visual studio](https://social.msdn.microsoft.com/Forums/en-US/78719729-9c69-4b00-9ae8-c2737f0000b0/catastrophic-failure-exception-from-hresult-0x8000ffff-eunexpected-visual-studio?forum=visualstudiogeneral)，但錯誤類型不相同，先列為參考

    - Azure Function 的文件上提到
        - 需要安裝 Azure Development Workload

            > 原本未完整安裝 Azure Development Workload

            ![7vdworkload](https://user-images.githubusercontent.com/3851540/48925453-9ee96580-eefe-11e8-8b96-0d533d2e9a2f.png)

        - 最新版的 Azure Function Tool

            > 原本版本為 Azure Function v2 Preview (.NET Standard)

            ![4afpreview](https://user-images.githubusercontent.com/3851540/48925450-9e50cf00-eefe-11e8-830b-4f9e75ffc8e3.png)

    - 更新 Visual Studio

        >原為 15.8.4

        ![3vsver](https://user-images.githubusercontent.com/3851540/48925449-9e50cf00-eefe-11e8-8f03-384491c9521b.png)

2. 實際解決方式

    - 升級 Visual Studio

        > 15.8.4 --> 15.9.2

        ![6vsver](https://user-images.githubusercontent.com/3851540/48925452-9ee96580-eefe-11e8-9b01-cf0473f90e96.png)
    - 升級 Azure Function Tool

        > Azure Function v2 Preview (.NET Standard) --> Azure Function v2 (.NET Core)

        ![5afnetcore](https://user-images.githubusercontent.com/3851540/48925451-9ee96580-eefe-11e8-9631-310987c8127e.png)

## 心得

從看到 fatal 錯誤訊息那刻，大概心裡就有底可能又是個奇案了：訊息本身沒有明確提出錯誤 module，甚至連 error code 都很隨便

果不其然，雖然解決方式很簡單但卻讓人搞不清背後真正的問題跟實際解決的做法，有點可惜，不過問題有解決就好可以繼續踩雷去了  哈哈

## 參考資訊

1. [catastrophic failure (exception from hresult 0x8000ffff (e_unexpected)) visual studio](https://social.msdn.microsoft.com/Forums/en-US/78719729-9c69-4b00-9ae8-c2737f0000b0/catastrophic-failure-exception-from-hresult-0x8000ffff-eunexpected-visual-studio?forum=visualstudiogeneral)
