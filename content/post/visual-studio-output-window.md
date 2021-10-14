---
title: "無法輸出訊息至 Visual Studio Output Window (輸出視窗)"
date: 2017-06-30T21:00:00+08:00
lastmod: 2021-10-14T21:00:15+08:00
draft: false
tags: ["Debug","Visual Studio"]
slug: "visual-studio-output-window"
aliases:
    - /2017/06/visual-studio-output-window.html
---
## 無法輸出訊息至 Visual Studio Output Window (輸出視窗)

同事想要 debug 某段程式碼，但不想中斷程式碼執行，並打算在執行過程中紀錄各個 method 實際執行的時間點。要符合同事的需求可以使用 log ，但如果透過 `System.Diagnostics.Debug` 或是 `System.Diagnostics.Trace` 來執行更簡單一點，不用安裝 log 套件也不需要設定

但同事使用上卻總是無法成功顯示 log 資訊，協助 debug 順手紀錄一下

## 關於 `Debug` 與 `Trace`

* 相同處
    1. 皆位於 `System.Diagnostics` 命名空間下
    2. 有相同方法
        * WriteLine
        * WriteLineIf
        * Indent
        * Unindent
        * Assert
        * Flush

    3. Debug Solution Configuration 時皆會輸出訊息
    4. 內容會輸出至 Visual Studio Output Windows 的 Debug 視窗中

        ![1outputdebug](https://user-images.githubusercontent.com/3851540/27722859-a66290d8-5d9c-11e7-9a25-da8ad6f3de40.png)

    5. 可以透過新增 Listener 將訊息寫至其他目標
        * Console
        * 檔案
        * nlog
        * .....其他

    6. Listener 是共用的

        > 無論新增 Listener 至 `Debug` or `Trace`，還是都會蒐集 `Debug` 與 `Trace` 訊息

* 相異處
    1. Release Solution Configuration 下，預設只有 `Trace` 內容會被輸出

        > 可以透過 project properties 來修改

        ![2enabledebug](https://user-images.githubusercontent.com/3851540/27722860-a684a6dc-5d9c-11e7-8d45-3996f6d91366.png)

## 如何使用？

1. 加入引用

    ```cs
    using System.Diagnostics;
    ```

2. 加上所需 log
    * Debug

        ```cs
        Debug.WriteLine("Test");
        ```

    * Trace

        ```cs
        Trace.WriteLine("Test");
        ```

## 檢查相關設定

1. 檢查訊息是輸出至 Output 視窗還是即時運算視窗

    * Visual Studio 主選單 Tools --> Options...

        ![3toolsoptions](https://user-images.githubusercontent.com/3851540/27722862-a6876160-5d9c-11e7-8d04-3fe55fa75d4c.png)

    * Debugging --> Redirect all output window text to the immediate window

        ![4redirect](https://user-images.githubusercontent.com/3851540/27722861-a6851b6c-5d9c-11e7-8c67-664fb3e5b497.png)

        * 想要在 Output 視窗看到訊息，請保持未選取的狀態

2. 檢查專案設定

    * 專案 上按右鍵 --> Properties

        ![5properties](https://user-images.githubusercontent.com/3851540/27722863-a6a4ca7a-5d9c-11e7-9ad3-9e00bffafa86.png)

    * 選擇所需 Configuration --> `Build` tab --> 確認 "Define DEBUG constant" 及 "Define TRACE constant" 狀態

        ![6buildconfig](https://user-images.githubusercontent.com/3851540/27722855-a65faa94-5d9c-11e7-8877-ba995ff8f072.png)

        * 預設 Debug configuariton ："Define DEBUG constant" 及 "Define TRACE constant" 都會勾選

            > 才可以使用 `Debug` 與 `Trace` 輸出訊息

        * 預設 Realease configuariton ：只有 "Define TRACE constant" 勾選

            > 才可以使用 `Trace` 輸出訊息

3. 檢查 Output 視窗設定

    * Output 視窗 --> Show output from：Debug

        ![7outputdebug](https://user-images.githubusercontent.com/3851540/27722858-a6618314-5d9c-11e7-9813-f594d106448d.png)

    * Output 視窗空白處按右鍵 --> 勾選 `Program output`

        ![7programoutput](https://user-images.githubusercontent.com/3851540/27722857-a6617d42-5d9c-11e7-81d8-07dfa5675524.png)

        * 預設所有選項都已勾選

            ![8allprogramoutput](https://user-images.githubusercontent.com/3851540/27722856-a66005a2-5d9c-11e7-84bf-e3032d99a731.png)

## 其他原因？

經過一番檢查確認後，同事還是無法正確地輸出訊息到 output 視窗，後來看到 web.config 有用了 [NLogTraceListener](http://nlog-project.org/2010/09/02/routing-system-diagnostics-trace-and-system-diagnostics-tracesource-logs-through-nlog.html)，猜測可能是 nlog 的設定問題，檢查 web.config

* 原始 web.config

    ```xml
    <system.diagnostics>
        <sharedListeners>
            <add name="nlog" type="NLog.NLogTraceListener, NLog" />
        </sharedListeners>
        <trace autoflush="true">
        <listeners>
            <add name="nlog" />
            <remove name="Default" />
        </listeners>
        </trace>
    </system.diagnostics>
    ```

* 需要移除 `<remove name="Default" />`

    > 這行設定將原本會將訊息輸出至 output 的行為移除了

    ```xml
    <system.diagnostics>
        <sharedListeners>
            <add name="nlog" type="NLog.NLogTraceListener, NLog" />
        </sharedListeners>
        <trace autoflush="true">
        <listeners>
            <add name="nlog" />
        </listeners>
        </trace>
    </system.diagnostics>
    ```

## 心得

這次除錯最大的心得就是團隊應該使用團隊成員有共識的專案設定及套件，避免其他同事接手時，為了小問題需要額外花費很多時間

## 參考資訊

1. [HOW TO： Visual C# .NET 中的 Trace 和 Debug 類別](https://support.microsoft.com/zh-tw/help/815788/how-to-trace-and-debug-in-visual-c)
2. [Debug.WriteLine shows nothing](https://stackoverflow.com/questions/9369820/debug-writeline-shows-nothing)
3. [NLogTraceListener](http://nlog-project.org/2010/09/02/routing-system-diagnostics-trace-and-system-diagnostics-tracesource-logs-through-nlog.html)
