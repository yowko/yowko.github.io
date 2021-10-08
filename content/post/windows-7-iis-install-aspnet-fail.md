---
title: "Windows 7 無法安裝 IIS ASP.NET 模組"
date: 2018-02-22T21:00:00+08:00
lastmod: 2021-10-08T21:00:37+08:00
draft: false
tags: ["IIS"]
slug: "windows-7-iis-install-aspnet-fail"
aliases:
    - /2018/02/windows-7-iis-install-aspnet-fail.html
---
## Windows 7 無法安裝 IIS ASP.NET 模組

這個問題發生在公司的 Windows 7 電腦上，一般日常的程式功能開發大多使用 IIS Express，用到 IIS 的機會並不高，但如果要同時開多個 website，IIS 使用的資源量就會比開多個 Visual Studio 降低很多

最近一直想要釐清同事遇到的 CORS 405 錯誤，在反覆測試的過程中竟然把 IIS 搞到無法 host ASP.NET application，身為 ASP.NET 開發人員，卻無法在 IIS 上建立 ASP.NET 站台XD，讓很多測試情境無法進行，嚴重影響工作效率

當然最終手段 - 重灌，可以解決問題，但除非必要實在不好意思麻煩 MIS 同事，幸虧最後找到不用重灌的解決方法，就來看看該如何解決問題吧

## 重現錯誤

1. 開啟 `Turn Windows features on or off`

    ![7turnfeature](https://user-images.githubusercontent.com/3851540/36530078-19c3f54a-17f5-11e8-9f9e-2883f98a526c.png)

2. Internet Information Services --> World Wide Web Services --> Application Development Features

    ![1winfeature](https://user-images.githubusercontent.com/3851540/36530072-18da143e-17f5-11e8-890a-5a04705f1e03.png)

3. 勾選 `ASP.NET` 會連帶選取 `.NET Extensibility`,`ISAPI Extensions`,`ISAPI Filters`

    ![2aspnet](https://user-images.githubusercontent.com/3851540/36530073-1901a63e-17f5-11e8-8063-f697d4d28fbc.png)

4. 開始安裝

    ![3installing](https://user-images.githubusercontent.com/3851540/36530074-19272eae-17f5-11e8-9cb9-b3288673f4d7.png)

5. 出現安裝失敗錯誤

    ![4fail](https://user-images.githubusercontent.com/3851540/36530075-1952e2a6-17f5-11e8-91b1-08e0b5bf5dba.png)

6. 要求重新開機

    ![5restart](https://user-images.githubusercontent.com/3851540/36530076-1978121a-17f5-11e8-8a1e-98331247d1ef.png)

## 錯誤訊息

* Event Viewer 中出現大量錯誤

    ![6eventviewer](https://user-images.githubusercontent.com/3851540/36530077-199e9e80-17f5-11e8-95f0-08652c566469.png)

* 錯誤訊息內容

    ```txt
    Unable to install counter strings because the SYSTEM\CurrentControlSet\Services\ASP.NET_2.0.50727\Performance key could not be opened or accessed. The first DWORD in the Data section contains the Win32 error code.
    ```

    ```txt
    Installing the performance counter strings for service ASP.NET_2.0.50727 (ASP.NET_2.0.50727) failed. The first DWORD in the Data section contains the error code.
    ```

    ```txt
    Faulting application name: aspnetca.exe, version: 7.5.7601.17855, time stamp: 0x4fc84161
    Faulting module name: aspnetca.exe, version: 7.5.7601.17855, time stamp: 0x4fc84161
    Exception code: 0xc0000005
    Fault offset: 0x0000000000023ccb
    Faulting process id: 0xc2af8
    Faulting application start time: 0x01d3ab005ada9d8b
    Faulting application path: C:\Windows\System32\inetsrv\aspnetca.exe
    Faulting module path: C:\Windows\System32\inetsrv\aspnetca.exe
    Report Id: 9993f40c-16f3-11e8-8d78-8851fb678dd9
    ```

    ```txt
    Unable to install counter strings because the SYSTEM\CurrentControlSet\Services\ASP.NET_64_2.0.50727\Performance key could not be opened or accessed. The first DWORD in the Data section contains the Win32 error code.
    ```

    ```txt
    Installing the performance counter strings for service ASP.NET_64_2.0.50727 (ASP.NET_64_2.0.50727) failed. The first DWORD in the Data section contains the error code.
    ```

    ```txt
    Unable to install counter strings because the SYSTEM\CurrentControlSet\Services\ASP.NET_64\Performance key could not be opened or accessed. The first DWORD in the Data section contains the Win32 error code.
    ```

    ```txt
    Installing the performance counter strings for service ASP.NET_64 (ASP.NET_64) failed. The first DWORD in the Data section contains the error code.
    ```

    ```txt
    Faulting application name: aspnetca.exe, version: 7.5.7601.17855, time stamp: 0x4fc8366d
    Faulting module name: aspnetca.exe, version: 7.5.7601.17855, time stamp: 0x4fc8366d
    Exception code: 0xc0000005
    Fault offset: 0x0001d950
    Faulting process id: 0xc287c
    Faulting application start time: 0x01d3ab0060a54ebf
    Faulting application path: C:\Windows\SysWOW64\inetsrv\aspnetca.exe
    Faulting module path: C:\Windows\SysWOW64\inetsrv\aspnetca.exe
    Report Id: 9eda2f9c-16f3-11e8-8d78-8851fb678dd9
    ```

    ```txt
    Unable to install counter strings because the SYSTEM\CurrentControlSet\Services\ASP.NET_2.0.50727\Performance key could not be opened or accessed. The first DWORD in the Data section contains the Win32 error code.
    ```

    ```txt
    Installing the performance counter strings for service ASP.NET_2.0.50727 (ASP.NET_2.0.50727) failed. The first DWORD in the Data section contains the error code.
    ```

## 解決方式

1. 移除 `Internet Information Services Hostable Web Core`
    * 開啟 `Turn Windows features on or off`

        ![7turnfeature](https://user-images.githubusercontent.com/3851540/36530078-19c3f54a-17f5-11e8-9f9e-2883f98a526c.png)

    * 取消勾選 `Internet Information Services Hostable Web Core` --> OK

        ![8iiswebcore](https://user-images.githubusercontent.com/3851540/36530079-19e8125e-17f5-11e8-8bc9-30abdceecae3.png)

2. 重新安裝 `Internet Information Services Hostable Web Core` 及 `ASP.NET`
    * 開啟 `Turn Windows features on or off`

        ![7turnfeature](https://user-images.githubusercontent.com/3851540/36530078-19c3f54a-17f5-11e8-9f9e-2883f98a526c.png)

    * 勾選 `Internet Information Services Hostable Web Core`

        ![9iiswebcore](https://user-images.githubusercontent.com/3851540/36530080-1a0d7a9e-17f5-11e8-9706-7b265bd6a676.png)

    * 勾選 `ASP.NET` 會連帶選取 `.NET Extensibility`,`ISAPI Extensions`,`ISAPI Filters`

        ![2aspnet](https://user-images.githubusercontent.com/3851540/36530073-1901a63e-17f5-11e8-8063-f697d4d28fbc.png)

## 心得

網路上也有其他網友分享可以手動加入 registry key，但我個人測試下未能真正解決問題，但 Event Viewer 所紀錄的錯誤數量確實有減少，如果有遇到相同問題試過解除安裝 `Internet Information Services Hostable Web Core` 無效後也許可以試試手動加入 registry key

也嘗過過重新註冊 asp.net ，前後遇到 `0x800702e4` (需要以 `administrator` 權限執行) 與 `0x80004005`，仍舊無法解決問題，個人經驗可以省略這個方式的嘗試

## 參考資訊

1. [Cannot enable .Net Extensibility and ASP.NET in Windows Features after VS installations](https://forums.asp.net/t/2001507.aspx?Cannot+enable+Net+Extensibility+and+ASP+NET+in+Windows+Features+after+VS+installations)
