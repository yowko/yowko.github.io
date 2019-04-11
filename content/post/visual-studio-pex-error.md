---
title: "Visual Studio 出現 CS0234 及 CS0246 - 找不到 'Pex' 的錯誤"
date: 2017-08-04T23:55:00+08:00
lastmod: 2018-09-24T00:02:12+08:00
draft: false
tags: ["Debug","Visual Studio"]
slug: "visual-studio-pex-error"
aliases:
    - /2017/08/visual-studio-pex-error.html
---
# Visual Studio 出現 CS0234 及 CS0246 - 找不到 'Pex' 的錯誤
同事 clone 專案之後，一 build 卻收到大量 error，馬上找出始作俑者 --> 我！！ 事不宜遲，立馬開始追查問題：

*   第一個疑問，這個專案在每次收到新的 push event 都會觸發 CI build，怎麼會同事 clone 後才發現
*   第二個疑問：因為公司政策問題，專案的參考一向都是包含所有 dll commit 的，缺漏 commit 檔案的機會不高
  

## 錯誤訊息

1.  訊息內容


    *   CS0234

        ```
        Severity Code Description Project File Line Suppression State
        Error CS0234 The type or namespace name 'Pex' does not exist in the namespace 'Microsoft' (are you missing an assembly reference?) DemoMSTest.IntelliTests C:\Users\YowkoTsai\Documents\Visual Studio 2015\Projects\Demo\DemoMSTest.IntelliTests\Properties\PexAssemblyInfo.cs 2 Active
        ```
    *   CS0246

        ```
        Severity Code Description Project File Line Suppression State
        Error CS0246 The type or namespace name 'PexAssemblySettingsAttribute' could not be found (are you missing a using directive or an assembly reference?) DemoMSTest.IntelliTests C:\Users\YowkoTsai\Documents\Visual Studio 2015\Projects\Demo\DemoMSTest.IntelliTests\Properties\PexAssemblyInfo.cs 9 Active
        ```
2.  錯誤畫面

    ![1error](https://user-images.githubusercontent.com/3851540/28976382-38e23096-7970-11e7-8cc4-cfae01f56716.png)

## 現有線索

1.  build error 出現在測試專案

    ![2testproject](https://user-images.githubusercontent.com/3851540/28976377-38965fc2-7970-11e7-8711-126d41e2692b.png)

2.  測試專案的 `PexAssemblyInfo.cs` 出現大量錯誤

    ![3pexassmerror](https://user-images.githubusercontent.com/3851540/28976379-38bc7608-7970-11e7-997b-6e3738070e14.png)

3.  測試專案遺失一個參考

    ![4missref](https://user-images.githubusercontent.com/3851540/28976380-38c612d0-7970-11e7-8b68-df0e7fa14e56.png)

## 發生原因推導

*   疑問一：為什麼有 CI 還會出現 build fail？

    > 目前該專案的 CI 只 build production code，測試專案並未包含在內，而這次出錯就是測試專案

*   疑問二：package 都 commit 了，為什麼會沒有參考？

    > 仔細查看我正常 build 的環境路徑，而同事的電腦上卻沒有這個檔案

    ![5pexpath](https://user-images.githubusercontent.com/3851540/28976381-38dbd8d6-7970-11e7-85de-15a6894b316c.png)

## 什麼是 `Microsoft.Pex.Framework`

從上面推導，大概可以確定是 `Microsoft.Pex.Framework` 造成的，但 `Microsoft.Pex.Framework` 是什麼勒？

*   **`Pex` 是一個自動產生具有高測試涵蓋率的套件**

    > 關於 `Pex` ，我在 Microsoft 網站並沒有找到較詳細的說明

*   如何加入專案參考的？

    > 透過上述描述(`自動`、`測試`)，大概可以知道應該與自動產生測試的功能有關，於是發現是 `IntelliTest` 造成的

    ![6intellitest](https://user-images.githubusercontent.com/3851540/28976383-38e5c4b8-7970-11e7-9fa7-18bfccfcdbb8.png)

*   為什麼同事電腦沒有

    > IntelliTest 的功能在 Visual Studio 2015 只有 Enterprise 版有，所以同事的電腦並沒有安裝到相關的 dll

*   解決方式

    > 解決方法很消極，但說實話 `IntelliTest` 的普及率及功能性好像沒有到非用不可的地步

    *   Copy 相關 dll
    *   移除 `IntelliTest` 測試

## 心得

這次的問題是我在建立測試時手殘，不小心選到 `Create IntelliTest` 造成的，雖然立馬就移除相關程式碼，但沒想到竟然有遺毒，還遇到 Visual Studio 版本不同造成的 dll 差異問題，好像應該要去扶老婆婆過馬路了 @@"

不過我覺得 Microsoft 這個設計有點問題，為什麼測試套件要相依於特定版本 Visual Studio 安裝路徑中的 dll，這樣沒有安裝正確版本的電腦都會出現問題，CI server 也會遇到一樣的狀況

# 參考資訊

1.  [Microsoft.Pex.Framework 命名空間](https://msdn.microsoft.com/zh-tw/library/microsoft.pex.framework.aspx)
2.  [Pex and Moles – Isolation and White Box Unit Testing for .NET](https://www.microsoft.com/en-us/research/project/pex-and-moles-isolation-and-white-box-unit-testing-for-net/)
