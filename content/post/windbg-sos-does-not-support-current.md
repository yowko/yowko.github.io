---
title: "WinDBG 出現 SOS does not support the current target architecture ?!"
date: 2018-01-28T02:00:00+08:00
lastmod: 2021-10-08T02:00:12+08:00
draft: false
tags: ["Debug","IIS","WinDBG"]
slug: "windbg-sos-does-not-support-current"
aliases:
    - /2018/01/windbg-sos-does-not-support-current.html
---
## WinDBG 出現 SOS does not support the current target architecture ?!

前幾天 production server 上的出現 CPU high 的 warning，當下立馬想到使用 WinDBG 來追查問題發生原因，只是我使用 WinDBG 的頻率不高，每次在使用前都要再花一些時間查語法，幸虧黑大日前的文章 - [WinDBG 應用實例：找出 ASP.NET CPU 100% 原因](http://blog.darkthread.net/post-2017-02-20-windbg-to-find-aspnet-cpu-high.aspx) 列舉幾個常用的 WinDBG 語法，讓使用 WinDBG 時的前置時間縮短不少，可以更有效率地專注在追查問題原因

以往使用 WinDBG 時都還算順利，但這次卻出現 `SOS does not support the current target architecture` 的狀況，讓追查動作無法繼續下去，所以立馬來紀錄一下這次使用 WinDBG 的經驗

## 操作流程

1. 下載並安裝 WinDBG

    > [Download Debugging tools for Windows](https://developer.microsoft.com/en-us/windows/hardware/download-windbg)

2. 開啟 WinDBG
    * 有 `32 位元` 及 `64 位元` 版本差異
    * 預設安裝位置在 `C:\Program Files (x86)\Windows Kits\10\Debuggers\` 資料夾下

        ![1windbgpath](https://user-images.githubusercontent.com/3851540/35474689-ba6a0238-03cc-11e8-94da-e845f1622d08.png)

3. 匯入 dump file (`.dmp`)
    * 主選單 `File` --> `Open Crash Dump...`

        ![2opendump](https://user-images.githubusercontent.com/3851540/35474690-ba928320-03cc-11e8-9b83-c05fc0f14460.png)

4. 依黑大文章 執行相關指令(以下指令內容節錄黑大文章內容)
    * `.sympath srv*D:\Symbol*https://msdl.microsoft.com/download/symbols`

        > 分析過程需要 Symbol 檔，指定 WinDbg 自動由微軟網站下載，並 Cache 在 D:\Symbol 目錄避免重複下載

        ![3symbolpath](https://user-images.githubusercontent.com/3851540/35474691-babb00ac-03cc-11e8-90f3-f454d5e4c3c2.png)

    * `!sym noisy`

        > 指定顯示完整 Symbol 下載資訊

        ![4symbolnoisy](https://user-images.githubusercontent.com/3851540/35474692-bae521e8-03cc-11e8-9011-35b781a46707.png)

    * `.cordll -ve -u -l`

        > 自動載入 CLR 偵錯相關模組

        ![5loaddll](https://user-images.githubusercontent.com/3851540/35474693-bb0ece08-03cc-11e8-8a22-bbaed8812917.png)

        * 用 64 位元 windbg 開 32 位元 dump file 就會出現找不到 dll 的錯誤

            ```txt
            Unable to load DLL mscordacwks_AMD64_x86_4.0.30319.18449.dll, Win32 error 0n87
            ```

            ![6loaddllerror](https://user-images.githubusercontent.com/3851540/35474694-bb3b09be-03cc-11e8-92c8-536fa056bf24.png)

    * `!runaway`

        > 依使用 CPU 時間將 thread 排序

        ![7runaway](https://user-images.githubusercontent.com/3851540/35474682-b93cbeaa-03cc-11e8-9665-15e3deb24396.png)

    * `~{thread id}s`

        > 切換至指定 thread

        ![8thread](https://user-images.githubusercontent.com/3851540/35474683-b966215a-03cc-11e8-9691-5d7035e50c17.png)

        * 如使用不正確的 WinDBG 版本 (32/64 位元) 開啟 .dump 會出現 `wow64cpu!CpupSyscallStub+0x2:`

            ![10wow64cpu](https://user-images.githubusercontent.com/3851540/35474685-b9bc75aa-03cc-11e8-870f-19c30d6935e5.png)

    * `!clrstack`

        > 列出該 thread 的 callstack

## 執行 `!clrstack` 時出現錯誤訊息

* 錯誤訊息內容

    ```txt
    SOS does not support the current target architecture.
    ```

* 錯誤訊息截圖

    ![9clrstack](https://user-images.githubusercontent.com/3851540/35474684-b98fd824-03cc-11e8-85f8-5fe7eabbcd60.png)

## 問題發生原因及解決方式

* 32 位元程式未使用 32 位元 task manager 匯出 dump file

  * 32 位元 task manager 位於 `C:\Windows\SysWOW64\Taskmgrexe`
  * 開啟後名稱為：`Task Manager (32 bit)`

      ![11taskmgr32bit](https://user-images.githubusercontent.com/3851540/35474686-b9e9e56c-03cc-11e8-99f9-c0cd8c2a92a1.png)

    * 同時僅能開啟一個 task manager，如已開啟 64 bit task manager 需先關閉才能開啟 32 bit task manager

* IIS - Application Pools 可以強制設定為 32 位元

    > 這就是我這次遇到狀況的實際源頭，現在多數 server 都是 64 位元，預設 IIS 也是使用 64 位元模式執行，但如果程式本身使用到僅支援 32 位元元件(ex. Oracle.DataAccess.x86) 就必需強制啟用 32 位元模式

  * IIS - Application Pools --> {Name} --> Advanced Settings

      ![12apppools](https://user-images.githubusercontent.com/3851540/35474687-ba13c936-03cc-11e8-858a-e7db144df4c1.png)

  * Enable 32-Bit Applications

      ![13enable32bit](https://user-images.githubusercontent.com/3851540/35474688-ba3e9ba2-03cc-11e8-9d9e-9105a02d864d.png)

## 心得

一開始想不通為什麼明明是 64-bit server 怎麼會出現 WinDBG 版本錯誤的訊息(`wow64cpu!CpupSyscallStub+0x2`) 而需要使用 x86 WinDBG 開啟，持續深入偵錯才又發現新的錯誤：`SOS does not support the current target architecture.` 才知道原來需要使用 32-bit task manager 匯出 dump file，經過一些實驗後終於證實是 application pool 設定的結果

經過這一連串的 debug 過程對於 WinDBG 的使用與指令都有更深的印象，非常感謝黑大文章 - [WinDBG 應用實例：找出 ASP.NET CPU 100% 原因](http://blog.darkthread.net/post-2017-02-20-windbg-to-find-aspnet-cpu-high.aspx) 幫了很大的忙，希望下次使用可以更得心應手

## 參考資訊

1. [WinDBG 應用實例：找出 ASP.NET CPU 100% 原因](http://blog.darkthread.net/post-2017-02-20-windbg-to-find-aspnet-cpu-high.aspx)
2. [Common WinDbg Commands (Thematically Grouped)](http://windbg.info/doc/1-common-cmds.html)
3. [Download Debugging tools for Windows](https://developer.microsoft.com/en-us/windows/hardware/download-windbg)
4. [SOS does not support the current target architecture](https://stackoverflow.com/questions/16422577/sos-does-not-support-the-current-target-architecture/16422887)
5. [Debugging Tools for Windows (WinDbg, KD, CDB, NTSD)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/index?WT.mc_id=DOP-MVP-5002594)
