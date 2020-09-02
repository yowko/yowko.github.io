---
title: "WinDbg 設定 symbol file path 的四種方法"
date: 2018-06-18T00:49:00+08:00
lastmod: 2018-10-07T00:49:42+08:00
draft: false
tags: ["Debug","WinDBG"]
slug: "windbg-symbol-file-path"
aliases:
    - /2018/06/windbg-symbol-file-path.html
---
# WinDbg 設定 symbol file path 的四種方法
有一陣子沒用 WinDbg 來進行偵錯，再次感受到年紀的影響，指令忘得很乾淨XD  當然 WinDbg 的指令對我而言本來就沒有記得很牢，忘得快也是意料中的事，剛好最近用到的機會高一些，每次查指令也滿花時間的，所以趁著連假時間做個紀錄以利之後追查問題可以再加快速度。

首先就來看看如何設定 symbol file path ，雖然 symbol file path 沒有特別設定也可以正常使用，不過每次都重新下載 symbol file 既耗時又浪費網路頻寬，因此透過設定 symbol file path 讓已經下載過的 symbol file 可以重用讓偵錯流程可以再加快，把握時間找出真正的問題


## symbol file path 格式
- 語法格式
    
    ```
    srv*{cache path}*{symbol server}
    ``` 
- 實際範例
    
    ```
    srv*D:\Symbol*https://msdl.microsoft.com/download/symbols
    ``` 

## A. 使用 WinDbg GUI
1. 主選單 `File` --> `Sympol File Path ...`
    
    ![1guisetting](https://user-images.githubusercontent.com/3851540/41510052-9295c476-7290-11e8-8097-641d3446410e.png) 
3. 在 `Symbol path` 填入
    
    ```
     srv*D:\Symbol*https://msdl.microsoft.com/download/symbols
    ``` 
    
    ![2symbolpath](https://user-images.githubusercontent.com/3851540/41510054-92caf59c-7290-11e8-8ae6-dfbf2dfd4ca0.png)

## B. 使用 CLI

```
windbg.exe -y srv*D:\Symbol*h
ttps://msdl.microsoft.com/download/symbols
```

![3cli](https://user-images.githubusercontent.com/3851540/41510055-930141b0-7290-11e8-8a94-4daf1cf0cd77.png)

## C. 使用 WinDbg 內建指令列
1. 主選單 `File` --> `Open Crash Dump ...`
    
    ![4opendump](https://user-images.githubusercontent.com/3851540/41510048-91625484-7290-11e8-8109-b60d4e237430.png) 
2. 執行下列指令
    
    ```
    .sympath srv*D:\Symbol*https://msdl.microsoft.com/download/symbols
    ``` 
    
    ![5sympath](https://user-images.githubusercontent.com/3851540/41510049-91e52f6c-7290-11e8-9815-34faa0ba2c4c.png)

## D. 使用環境變數

> 使用 `SETX` 指令

```
SETX _NT_SYMBOL_PATH srv*D:\Symbol*https://msdl.microsoft.com/download/symbols
```

![6setx](https://user-images.githubusercontent.com/3851540/41510050-9226708a-7290-11e8-91a0-6b88232671b0.png)

## 心得
設定完成後可以在 WinDbg 的 command prompt 中執行 `!sympath` 確認使用中的 symbol path

![7sympathresult](https://user-images.githubusercontent.com/3851540/41510051-9261fcf4-7290-11e8-927d-59e23c1ff2bf.png)


# 參考資訊
1. [Setting Symbol and Executable Image Paths in WinDbg](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/setting-symbol-and-source-paths-in-windbg?WT.mc_id=DOP-MVP-5002594)
2. [Symbol path for Windows debuggers](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-path?WT.mc_id=DOP-MVP-5002594)
3. [Setting and getting windows environment variables from the command prompt?](https://superuser.com/questions/79612/setting-and-getting-windows-environment-variables-from-the-command-prompt)