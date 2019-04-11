---
title: "DebugDiag 2 Analysis 出現 DacError"
date: 2018-01-28T22:43:00+08:00
lastmod: 2018-10-03T22:43:49+08:00
draft: false
tags: ["Debug","IIS","WinDBG"]
slug: "debugdiag-2-analysis-dacerror"
aliases:
    - /2018/01/debugdiag-2-analysis-dacerror.html
---
# DebugDiag 2 Analysis 出現 DacError
繼前篇筆記 [WinDBG 出現 SOS does not support the current target architecture ?!](https://blog.yowko.com/2018/01/windbg-sos-does-not-support-current.html) 紀錄在使用 WinDBG 偵錯 production issue 時遇到使用 64-bit 工作管理員匯出 32-bit 程式的 dump 而無法順利完成偵錯的狀況

遇到問題當下並沒有其他心思去偵錯 WinDBG 所以立馬改使用黑大在 [ASP.NET CPU 飆高問題之傻瓜分析工具－DebugDiag Tools](http://blog2.darkthread.net/blogs/darkthreadtw/archive/2017/02/22/13021.aspx) 一文中介紹的 `Debug Diagnostic Tool (DebugDiag)` 來查問題，但第一次使用 Debug Diagnostic Tool (DebugDiag)，加上問題源頭原本就與工具無關，並沒有順利查出問題根源

在使用 `Debug Diagnostic Tool (DebugDiag)` 過程遇到不常見的錯誤訊息，於是紀錄一下使用經驗

## 操作步驟

1.  下載並安裝 `Debug Diagnostic Tool` (最新版為：[Debug Diagnostic Tool v2 Update 2](https://www.microsoft.com/en-us/download/details.aspx?id=49924))
    *   有 x86 與 x64 版本，請依執行環境選擇

        ![1x86x64](https://user-images.githubusercontent.com/3851540/35483248-d5531356-047a-11e8-9718-9d2c243375d3.png)

    *   Dump Analysis 是分析工具也是本文的主角

        ![2dumpanalysis](https://user-images.githubusercontent.com/3851540/35483249-d57f6410-047a-11e8-8bc7-634cc8641c47.png)

2.  開啟 `DebugDiag 2 Analysis` 並 匯入 dump file
    *   搜尋 `DebugDiag 2 Analysis`

        ![4search](https://user-images.githubusercontent.com/3851540/35483252-d5d65d4c-047a-11e8-85a2-7e7b6125eaa5.png)

    *   Add Data Files

        ![3addfile](https://user-images.githubusercontent.com/3851540/35483251-d5ac07ea-047a-11e8-9d0d-6cd1840816cc.png)

3.  選擇分析規則
    *   PerAnalysis

        ![5peranalysis](https://user-images.githubusercontent.com/3851540/35483253-d5fffd00-047a-11e8-8a3d-a3879a9a7552.png)

4.  執行分析(Start Analysis)

    ![6startanalysis](https://user-images.githubusercontent.com/3851540/35483254-d62acd00-047a-11e8-8522-29934a62d674.png)

    *   分析報告為 `.mht` 檔，請使用 IE 開啟效果跟版面才會正常
        *   IE

            ![7ie](https://user-images.githubusercontent.com/3851540/35483255-d6531382-047a-11e8-951a-7d7fe183d9ce.png)

        *   Chrome (基本的圖形跟連結都無法使用)

            ![8chrome](https://user-images.githubusercontent.com/3851540/35483256-d67b95f0-047a-11e8-8a14-ad46867e6cd2.png)

## 分析報告出現錯誤警示

> 我嘗試匯入兩份 dump file，都得到 dac 異常的錯誤，但錯誤細節內容不同

*   警示訊息內容


    *   dump 1

        ```
        Analysis results may be incomplete because an error occurred while initializing the CLR diagnostic runtime for w3wp.DMP.
            
        Dump File:  w3wp.DMP
        
        Type:  Microsoft.Diagnostics.Runtime.ClrDiagnosticsException
        
        Message:  Failure loading DAC: CreateDacInstance failed 0x8000ffff
        
        Stack Trace:
        Microsoft.Diagnostics.Runtime.DacLibrary..ctor(DataTargetImpl dataTarget, String dacDll)
        Microsoft.Diagnostics.Runtime.DataTargetImpl.CreateRuntime(String dacFilename)
        DebugDiag.DotNet.NetDbgObj.CreateRuntime(String symbolPath, DataTarget target, Int32 runtimeIndex, ClrInfo& clrInfo)
        DebugDiag.DotNet.NetDbgObj.CreateRuntimeAndGetHeap(String dumpPath, IDbgObj3 legacyDebugger, String symbolPath, Boolean throwOnBitnessMismatch, Boolean loadClrHeap)
            
        HResult: DacError
        ```

    *   dump 2

        ```
        Analysis results may be incomplete because an error occurred while initializing the CLR diagnostic runtime for 199w3wp.DMP.
            
        Dump File:  199w3wp.DMP
        
        Type:  DebugDiag.DotNet.DacNotFoundException
        
        Message:  CLR is loaded in the target, but the correct dac file cannot be found. DacFileName: mscordacwks_X86_Amd64_4.0.30319.18449.dll. DacLocation:
        ```
*   警示訊息截圖
    *   dump 1

        ![9error1](https://user-images.githubusercontent.com/3851540/35483257-d6cc27d6-047a-11e8-8c7d-37e1a28bd672.png)

    *   dump 2

        ![10error2](https://user-images.githubusercontent.com/3851540/35483258-d6f46e80-047a-11e8-9e51-f45cdd49088c.png)

## 問題發生原因以及解決方式

> 發生原因與 [WinDBG 出現 SOS does not support the current target architecture ?!](https://blog.yowko.com/2018/01/windbg-sos-does-not-support-current.html) 相同，皆是 32-bit 程式未使用 32-bit task manager 匯出 dump file 引起

*   32 位元程式未使用 32 位元 task manager 匯出 dump file
    *   32 位元 task manager 位於 `C:\Windows\SysWOW64\Taskmgr.exe`
    *   開啟後名稱為：`Task Manager (32 bit)`

        ![11taskmgr32bit](https://user-images.githubusercontent.com/3851540/35474686-b9e9e56c-03cc-11e8-99f9-c0cd8c2a92a1.png)

    *   同時僅能開啟一個 task manager，如已開啟 64 bit task manager 需先關閉才能開啟 32 bit task manager

*  IIS - Application Pools 可以強制設定為 32 位元

    > 這就是我這次遇到狀況的實際源頭，現在多數 server 都是 64 位元，預設 IIS 也是使用 64 位元模式執行，但如果程式本身使用到僅支援 32 位元元件(ex. Oracle.DataAccess.x86) 就必需強制啟用 32 位元模式

    *   IIS - Application Pools --> {Name} --> Advanced Settings

        ![12apppools](https://user-images.githubusercontent.com/3851540/35474687-ba13c936-03cc-11e8-858a-e7db144df4c1.png)

    *   Enable 32-Bit Applications

        ![13enable32bit](https://user-images.githubusercontent.com/3851540/35474688-ba3e9ba2-03cc-11e8-9d9e-9105a02d864d.png)

## 心得

同時使用 `WinDBG` 與 `DebugDiag 2 Analysis` 下，`DebugDiag 2 Analysis` 明顯比 `WinDBG` 好上手許多，使用上也沒有 x86 兆 x64 限制，加上不用記指令，非常方便

不過 `DebugDiag 2 Analysis` 產出的報表卻只有 IE 能開？！ 使用 edge 開啟會提示是否要開啟 `.mht` 接著透過 IE 開啟，Chrome 還直接大跑版，雖然體驗沒那麼好，但還能接受，畢竟本來 UI/UX 就不是 debug production issue 的重點

不過有一點倒是 `WinDBG` 勝過 `DebugDiag 2 Analysis` 不少，就是關於使用錯誤 task manager 匯出 dump file 的錯誤提示易讀性：

*   `WinDBG`：SOS does not support the current target architecture
*   `DebugDiag 2 Analysis`：Analysis results may be incomplete because an error occurred while initializing the CLR diagnostic runtime for w3wp.DMP


要不是已經知道問題發生原因，不然光看錯誤訊息還真不知道該如何查起XD

# 參考資訊

1.  [ASP.NET CPU 飆高問題之傻瓜分析工具－DebugDiag Tools](http://blog2.darkthread.net/blogs/darkthreadtw/archive/2017/02/22/13021.aspx)
2.  [WinDBG 出現 SOS does not support the current target architecture ?!](https://blog.yowko.com/2018/01/windbg-sos-does-not-support-current.html)
3.  [Debug Diagnostic Tool v2 Update 2](https://www.microsoft.com/en-us/download/details.aspx?id=49924)
4.  [線上網站很慢！使用DebugDiagnostic Tool進行線上IIS網站程式效能分析](http://blog.kkbruce.net/2017/09/use-debugdiagnostic-tool-dump-online-iis-appliaction-problem.html)
