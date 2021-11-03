---
title: "如何看程式是 32 bit 還是 64 bit"
date: 2017-11-18T23:51:00+08:00
lastmod: 2021-11-03T23:51:59+08:00
draft: false
tags: ["Oracle","Tools"]
slug: "binary-is-32-bit-64-bit"
aliases:
    - /2017/11/binary-is-32-bit-64-bit.html
---
## 如何看程式是 32 bit 還是 64 bit

今天正在嘗試使用 docker 安裝 oracle client，嘗試的過程中發現之前的筆記有錯誤的地方：[Dapper 讀取 Oracle 資料](/2017/07/dapper-oracle.html) 文中提到 Oracle.ManagedDataAccess 是 ~~64 bit 元件，也無法正確連線 oracle~~，兩項說明都是錯誤的， Oracle.ManagedDataAccess 是 `32 bit`，而且`絕對可以用來操作 oracle`

在釐清與驗證之前筆記到底是不是寫錯的過程中有兩個重點：

1. Oracle.ManagedDataAccess 是 32 bit 還是 64 bit ？
2. Oracle.ManagedDataAccess 到底能否可以用來操作 oracle ？

結果就如前面提到的：筆記內容錯了，我道歉 @@"，修正後的內容待調整完整再補充，在完成前先來分享該如何檢查程式 (.dll or .exe) 是 32 bit or 64 bit

## 使用 `dumpbin.exe`

> 這個工具會隨著 Visual Studio 安裝，是 Visual C++ 所屬的工具之一

1. 位置如下

    > 實際位置與 Visual Studio 版本有關

    ```cmd
    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64_x86\dumpbin.exe
    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64_arm\dumpbin.exe
    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\dumpbin.exe
    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\x86_arm\dumpbin.exe
    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\x86_amd64\dumpbin.exe
    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\dumpbin.exe
    ```

    ![1location](https://user-images.githubusercontent.com/3851540/32982135-93b8c0be-ccba-11e7-9934-252767f5f906.png)

2. 如何使用 `dumpbin.exe`

    * 格式

        ```cmd
        dumpbin.exe /headers {檔案}
        ```

    * 範例

        ```cmd
        "C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\dumpbin.exe" /headers Oracle.ManagedDataAccess.dll
        ```

3. FILE HEADER 提供相關資訊

    * 確認 `Oracle.ManagedDataAccess` 確實為 32 bit 組件

        ![2Oracle.ManagedDataAccess](https://user-images.githubusercontent.com/3851540/32982136-940b950a-ccba-11e7-91ad-323639f77516.png)

    * 64 bit 對照組

        > 這是自己設定組譯成 64 bit，用來對照參考用

        ![364](https://user-images.githubusercontent.com/3851540/32982137-9434004e-ccba-11e7-863d-7541cde8b22e.png)

## 心得

心裡一直希望不是之前筆記寫錯，但最後結果確認真的寫錯，雖然 Oracle 不是我熟悉的內容，不過我相信沒有經過這次驗證的過程，問我 SQL Server 用的 System.Data.SqlClient 是 32 bit 還是 64 bit 我一樣是答不出來的，只是內容寫錯心裡還是不太舒服，只能安慰自己下次多注意、也是透過這次經驗才學到怎麼看 32/64 bit 的

## 參考資訊

1. [Dapper 讀取 Oracle 資料](/2017/07/dapper-oracle.html)
2. [DUMPBIN 參考](https://msdn.microsoft.com/zh-tw/library/c1h23y6c.aspx)
3. [Windows command to tell whether a .dll file is 32 bit or 64 bit?](https://stackoverflow.com/questions/14560866/windows-command-to-tell-whether-a-dll-file-is-32-bit-or-64-bit)
