---
title: "使用 MethodTimer.Fody 來為 method 加上時間測量"
date: 2024-11-27T00:30:00+08:00
lastmod: 2024-11-27T00:30:31+08:00
draft: false
tags: ["dotnet","csharp","benchmark"]
slug: "measure-method-performance-with-methodtimer-fody"
---

## 使用 MethodTimer.Fody 來為 method 加上時間測量

之前筆記 [Stopwatch 的正確用法](/stopwatch-best-practice) 紀錄到如何使用 Stopwatch 的新 API，讓我想起過去常使用的 [GitHub:MethodTimer.Fody](https://github.com/Fody/MethodTimer) 也是透過 `Stopwatch.StartNew()` 方式來計算時間，回頭確認是否有更新時，才想起沒有為 `MethodTimer.Fody` 寫過筆記，畢竟它的用法簡單，不需要太多的說明。剛好因為 Stopwatch 新 API 的關係想連帶紀錄一下其他東西，就趁這個機會就來紀錄一下 `MethodTimer.Fody` 的使用方式。

## 基本環境說明

- macOS Sequoia 15.1.1 (Apple M2 Pro)
- JetBrains Rider 2024.3
- dotnet SDK 9.0.100
- NuGet package

    - MethodTimer.Fody 3.2.2

## 使用方式

1. 安裝 NuGet package

    ```bash
    dotnet add package MethodTimer.Fody
    ```

2. 加入 `FodyWeavers.xml` 檔案 (NuGet 安裝後會自動產生)

    {{<gist yowko 1cb84abc32472a7e7973212d853af96e "FodyWeavers.xml">}}

3. 新增 Interceptor (攔截器)

    > 比照套件原生方式使用 trace 來記錄執行時間

    {{<gist yowko 1cb84abc32472a7e7973212d853af96e "MethodTimeLogger.cs">}}

    - 如果沒有自訂 Interceptor，預設會使用 `Trace.WriteLine` 來紀錄內容，以 ASP.NET Core 為例，可以新增一個 ConsoleTraceListener 做為 TraceListener 的顯示方式

        {{<gist yowko 1cb84abc32472a7e7973212d853af96e "ProgramTracer.cs">}}

4. 將 ASP.NET 用的 logger 設定為 MethodTimer.Fody

    {{<gist yowko 1cb84abc32472a7e7973212d853af96e "Program.cs" >}}

5. 在想要測量執行時間 method 上方加入 `Time` attribute

    > 可以視需求來決定是否要紀錄參數或是其他訊息

    - 無參數的 method

        {{<gist yowko 1cb84abc32472a7e7973212d853af96e "WeatherForecastController.cs">}}

    - 有參數的 method

        {{<gist yowko 1cb84abc32472a7e7973212d853af96e "WeatherForecastControllerParameter.cs">}}

6. 修改 log level 至 `Trace` 以顯示我們在 Interceptor 中設定的 log

    {{<gist yowko 1cb84abc32472a7e7973212d853af96e "appsettings.json">}}

7. 實際效果

    - 無參數

        ![nopara](https://github.com/user-attachments/assets/cd2e0892-e05c-4261-815b-ab09d3d5c4bf)

    - 有參數

        ![para](https://github.com/user-attachments/assets/3161870e-19f4-4445-8802-e9d02bd2f9b1)

## 心得

[GitHub:MethodTimer.Fody](https://github.com/Fody/MethodTimer) 提供了非常簡單的方式：在 method 上加上 `Time` attribute 來計算 method 的執行時間，另外可以自訂 Interceptor 來客製記錄的內容與方式，非常方便

不過 MethodTimer.Fody 是基於 Fody 的套件，Fody 是一個編譯後工具（Post-Compiler Tool）。它修改的是已生成的 IL，也就是在 .NET 程式集已經生成後進行操作，看不到插入的程式碼，如果有 debug 的需求，難度較高

回到 Stopwatch 新 API 的議題，我看 MethodTimer.Fody 已經將修改 commit 至 [struct-stopwatch](https://github.com/Fody/MethodTimer/tree/struct-stopwatch?tab=readme-ov-file) branch，但一直沒有 merge 至 master branch，所以目前還是使用 `Stopwatch.StartNew()` 的方式來計算時間。

## 參考資料

1. [Stopwatch 的正確用法](/stopwatch-best-practice)
2. [GitHub:MethodTimer.Fody](https://github.com/Fody/MethodTimer)
3. [GitHub:MethodTimer.Fody:struct-stopwatch](https://github.com/Fody/MethodTimer/tree/struct-stopwatch?tab=readme-ov-file)
4. [Youtube:The Easiest Way to Measure Your Method's Performance in C#](https://www.youtube.com/watch?v=xlqcT4NSrZw)
