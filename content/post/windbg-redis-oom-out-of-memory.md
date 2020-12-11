---
title: "使用 WinDbg 查出 Redis OOM - Out of Memory"
date: 2018-07-16T15:41:00+08:00
lastmod: 2020-12-11T15:41:07+08:00
draft: false
tags: ["Debug","Redis","WinDBG"]
slug: "windbg-redis-oom-out-of-memory"
aliases:
    - /2018/07/windbg-redis-oom-out-of-memory.html
---
# 使用 WinDbg 查出 Redis OOM - Out of Memory
Session 是網站開發時既方便又相對 Cookie 安全的資料儲存技術，但 Session 弱型別的特性容易造成程式碼可讀性降低，另外 Session 將資料儲存在 server 上的特性也會影響 server 效能

Session 的優劣相信大家心中各有自己的標準，沒有絕對一致的看法，只是以我個人而言近幾年的新專案中已大幅降低了 Session 的使用，主要就是希望可以提高程式碼的可讀性

公司有部份網站使用 Session 來儲存資料，加上希望跨多台機器存取而利用 Redis 來當做儲存媒介，最近公司的大型活動讓 user 來訪數大增，也讓 Redis 用量超出預期而出現錯誤，剛好利用這次機會紀錄一下使用 WinDbg 找到 Redis OOM - Out Of Memory 的過程

## WinDbg debug 步驟
1. 取得正確 dump 檔
    
    > 在使用 WinDbg debug 時需要特別注意 32 bit |64 bit 的應用程式有不同的使用方式 ： 32 bit 應用程式就得透過 32 bit 的 taskmgr 來建立 dump 也只能用 x86 的 WinDbg 來進行分析，詳情可以參考 [WinDBG 出現 SOS does not support the current target architecture ?!](/2018/01/windbg-sos-does-not-support-current.html)
2. 使用正確架構的 WinDbg
    
    >WinDbg 有兩種版本：`x86`、`x64`，請依實際 application 的架構來選用
    
    ![1windbg](https://user-images.githubusercontent.com/3851540/42746503-a63006c8-890a-11e8-9324-f2ca0fcd5ffc.png)
3. 開啟 dump file
    
    > File --> Open Crash Dump...

    ![2opendump](https://user-images.githubusercontent.com/3851540/42746504-a658d846-890a-11e8-9f76-b35bd3b02831.png) 
4. 設定 symbol 路徑 (optional)
    
    > 可以為 WinDbg 設定儲存 symbol 的暫存路徑避免重複下載，詳情可以參考 [WinDbg 設定 symbol file path 的四種方法](/2018/06/windbg-symbol-file-path.html)
5. 載入 CLR 偵錯模組
    
    > .cordll -ve -l，詳細資訊請參考 [.cordll (Control CLR Debugging)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-cordll--control-clr-debugging-?WT.mc_id=DOP-MVP-5002594)
    
    - `-ve` ：開啟詳細資訊
    - `-l` ：載入 CLR debug 模組
    
    ![3cordll](https://user-images.githubusercontent.com/3851540/42746498-a5882b6a-890a-11e8-8af8-c9dd59f11397.png) 
6. 列出所有錯誤
    
    >!dumpheap -type Exception -stat
    
    - `-type`：過濾 heap 的型別
    - `-stat`：僅列出統計資訊

    ![4dumpheap](https://user-images.githubusercontent.com/3851540/42746501-a5d90fc6-890a-11e8-8921-78524375dd35.png)
7. 列出所有 exception 記憶體位置
    
    > !DumpHeap -mt 1a6e01a4
    
    - `-mt`：指定 `MethodTable` 為上一個步驟中列出的 `MT`
    
    ![5dumpheapmt](https://user-images.githubusercontent.com/3851540/42746500-a5b08eac-890a-11e8-8ca1-e36598a3f3af.png)
8. 檢視 exception 訊息
    
    >!pe 3d4652b4 
    
    ```
    ERR Error running script (call to f_71fe2e7962348b06aa8ce3e244cdb3f774b4f549): @user_script:4: @user_script: 4: -OOM command not allowed when used memory > 'maxmemory'. 
    ```

    ![6pe](https://user-images.githubusercontent.com/3851540/42746502-a6031082-890a-11e8-80f7-4d9d40c4e2e2.png)

## 心得
當時 production 上出現這個問題時，造成大量 request 被 queue 住無法正確 response 而嚴重影響 user 操作，所幸即時找到問題主因沒讓事勢持續惡化

不過這次的狀況也突顯出環境設定的重要性，一個簡單的設定就足以影響整個 application 的生死，如果對相關 component 的設定沒有足夠掌握度很容易讓好的工具或是架構蒙受不白之冤呀

# 參考資訊
1. [使用 Redis 當做 ASP.NET MVC 的 Session State Server](/2017/01/redis-aspnet-mvc-session-state-server.html)
2. [WinDBG 出現 SOS does not support the current target architecture ?!](/2018/01/windbg-sos-does-not-support-current.html)
3. [WinDbg 設定 symbol file path 的四種方法](/2018/06/windbg-symbol-file-path.html)
4. [.cordll (Control CLR Debugging)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-cordll--control-clr-debugging-?WT.mc_id=DOP-MVP-5002594)
5. [Redis with Resque and Rails: ERR command not allowed when used memory > 'maxmemory'](https://stackoverflow.com/questions/9987832/redis-with-resque-and-rails-err-command-not-allowed-when-used-memory-maxmemo)