---
title: "隨 .NET9 SDK 推出的 C#13 新增 Lock 類別與用法"
date: 2024-11-07T00:30:00+08:00
lastmod: 2024-11-07T00:30:31+08:00
draft: false
tags: ["dotnet","csharp"]
slug: "dotnet9-csharp13-lock"
---

## 隨 .NET9 SDK 推出的 C#13 新增 Lock 類別與用法

在 C# 中，lock 語法確保程式碼區塊的執行不會被其他 thread 影響。過去 lock 語法就是 System.Threading.Monitor 的語法糖，編譯器產生的程式碼等於使用 Monitor.Enter 和 Monitor.Exit 來管理 thread 對關鍵部分的執行

.NET9 SDK 推出的 C#13 加入新的 System.Threading.Lock 類型。這意味著，當您使用 Lock 物件作為 lock 語法的目標時，編譯器會產生使用 `Lock.EnterScope()`方法進入獨佔範圍並在結束時使用 `Dispose()` 方法退出範圍的程式碼，反之則會建立基於 System.Threading.Monitor 傳統 API 的程式碼

最棒的是只要更改物件類型 (由 object 改為 Lock) 就可以由 Monitor 改為 Lock，不用額外手動修改程式碼。

## 基本環境說明

- macOS Sequoia 15.0.1 (Apple M2 Pro)
- dotnet sdk v9.0.100-rc.2

## 使用方式

1. 使用 object (舊語法)

    - lock object 語法糖

        {{<gist yowko 19def5bc8a9a8a911a9eb4ec14f49321 "oldlock.cs">}}

    - 底層是 Monitor.Enter 和 Monitor.Exit

        {{<gist yowko 19def5bc8a9a8a911a9eb4ec14f49321 "oldlock_rewrite.cs">}}

2. 新語法

    - 使用 Lock 類型

        {{<gist yowko 19def5bc8a9a8a911a9eb4ec14f49321 "newlock.cs">}}

    - 使用 using

        {{<gist yowko 19def5bc8a9a8a911a9eb4ec14f49321 "newlock_using.cs">}}

    - 底層

        > 兩者的底層程式碼是相同的

        {{<gist yowko 19def5bc8a9a8a911a9eb4ec14f49321 "newlock_rewrite.cs">}}

## 心得

System.Threading.Lock 類型的優勢：

1. 更好的性能：
    System.Threading.Lock 比傳統的 Monitor 更高效，從而在多線程應用程式中實現更好的性能。
    每個 C# 物件的標頭中都有一個 4 byte 的 field（稱為 SyncBlock），用於鎖定，該欄位還用於儲存物件的雜湊碼，需要同時執行這兩項操作時，就不免有額外的程式碼。而這個新的 Lock 單純只是 lock 沒有這個 field 而使鎖定代碼稍微簡單一些。
2. 更清晰的語法：System.Threading.Lock 提供了更清晰、更直接的 API，使開發人員更容易實現正確的同步。
3. 健壯的錯誤處理：通過強制實施 Dispose（） 模式，System.Threading.Lock 有助於防止常見錯誤，例如忘記釋放鎖。

`Lock` 與 `Using` 兩者會產生相同的 IL 程式碼，使用時機我覺得可以從下面幾點考量

1. 如果是舊有的程式碼升級上來，直接用 `Lock` 改動會少點
2. `Lock` 比 `Using` 語意更直接也更加直覺
3. `Using` 則是適合更想精準控制釋放 lock 時機的情境

## 參考資訊

1. [Microsoft Learn:What's new in C# 13](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13#new-lock-object?WT.mc_id=DOP-MVP-5002594)
2. [Microsoft Learn:Lock object](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-13.0/lock-object?WT.mc_id=DOP-MVP-5002594)
3. [System.Threading.Lock: A New Thread Synchronization in .NET 9](https://thetechplatform.medium.com/system-threading-lock-a-new-thread-synchronization-in-net-9-cf850339b2dc)
4. [Lock keyword gets an upgrade in .NET9](https://mareks-082.medium.com/new-lock-object-and-history-d69877f46521)
5. [GitHub:dotnet/runtime - Add first class System.Threading.Lock type](https://github.com/dotnet/runtime/issues/34812)
