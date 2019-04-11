---
title: "ProcDump - 用來建立執行程式的 dump"
date: 2017-03-20T02:43:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "procdump"
aliases:
    - /2017/03/procdump.html
---
# ProcDump - 用來建立執行程式的 dump
這是 Sysinternal 製作的好用工具，用來建立 dump 應該沒有其他更好用的 command line 工具了吧？！是我太久沒用了嗎？  我怎麼覺得參數功能變多了，使用過程隨手筆記一下，如果有錯請大家指正呀


## 用法

```
procdump [-a] [[-c|-cl CPU usage] [-u] [-s seconds]] [-n exceeds] [-e [1 [-b]] [-f <filter,...>] [-g] [-h] [-l] [-m|-ml commit usage] [-ma | -mp] [-o] [-p|-pl counter threshold] [-r] [-t] [-d <callback DLL>] [-64] <[-w] <process name or service name or PID> [dump file] | -i <dump file> | -u | -x <dump file> <image file> [arguments] >] [-? [ -e]
```

## 參數說明

參數|說明
:---|:---
-a|	避免中斷.需配合 `-r` 使用. 建立 dump 時可能因為超過建立 dump 程序數量會造成目標程式長時間無法回應時，這個建立 dump 的要求會被忽略
-b|	將 debug 中斷點視為異常 (如果未加這個參數時則是忽略).
-c|	CPU 使用率高於指定門檻自動建立 dump 檔
-cl|CPU 使用率低於指定門檻自動建立 dump 檔
-d|	執行叫做 `MiniDumpCallbackRoutine` 的 dll 當作 miniudmp 的 callback
-e|	程式發生未預期錯誤時自動建立 dump 檔，包含第一個自動建立 dump 檔出現錯誤
-f|	只處理第一個發生的異常. 支援萬用字元 (*) .可以使用 空白過濾器("") 來顯示名稱而不建立 dump.過濾第一次機會異常。 支持通配符（*）。 要僅顯示名稱而不轉儲，請使用空白（「」）過濾器。
-g|	以原始 debugger 執行在 managed process 中 (無互動操作).
-h|	為沒有回應的程式建立 dump(未回應超過 5 秒).
-i|	將 ProcDump 當做 AeDebug 事後 debugger. 只能搭配 `-ma`, `-mp`, `-d` ,`-r` 一起使用
-l|	顯示程式的 debug log
-m|	Memory 使用量高於指定 MB 數時建立 dump
-ma|建立完整 dump，預設只有 thread 跟 handle 資訊
-ml|Memory 使用量下降至於指定 MB 數時建立 dump
-mp|將 thread , handle 及所有 memory 的讀寫資訊寫入，為了最小化 dump 的尺寸，如果有超過 512MB 的 memory 區域就會排除最 memory 區域.相同大小的 memory allocation 會放在同個 meomry 區域 . 對於 Exchange and SQL Server 刪除 memory cache 可以減少 90% dump 
-n|	在程式結束前要建立幾個 dump 檔
-o|	實際使用：指定 dump 檔存放位置 英文：Overwrite an existing dump file.
-p|	performance counter 超過指定門檻時會啟動. Note: 在多 cpu 環境下指定程式計數器需使用特定語法：`\Process(<name>_<pid>)\counter`。 
-pl|erformance counter 低於指定值時觸發建立 dump
-r|使用 clone 模式來建立 dump。 Concurrent 限制是可自訂的（default：1，最大：5）。 注意：高 Concurrent 值可能會影響系統性能。<br/>Windows 7：使用反射。 OS 不支持 `-e`。<br/>Windows 8.0：使用反射。 OS 不支持 `-e`。<br/>Windows 8.1+：使用PSS。 支持所有觸發類型。
-s|	指定 CPU 使用到達門檻維持秒數才建立 dump (預設是 10 秒).
-t|	當程式終止就建立 dump
-u|	將 CPU 使用率視為單核需搭配 `-c` 一起使用）。如果是唯一的參數，取消 ProcDump 作為事後 debugger
-w|	如果指定程式未執行中，等待它啟動。
-x|	使用可選參數啟動指定的 image.如果它是 Application or Package，ProcDump 將只會在下次啟動時執行
-64|ProcDump 預設建立 32-bit 的 dump，可以使用這個參數來建立 64-bit 的 dump
-?|	使用 `-？ -e` 查看範例指令。
-accepteula|預設接受 Sysinternal lincense 協議。

* 如果省略 dump file ，預設為 {processname}_{datetime}.dmp。

# 參考資訊
1. [ProcDump v8.2](https://technet.microsoft.com/en-us/sysinternals/dd996900.aspx)