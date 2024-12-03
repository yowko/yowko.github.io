---
title: "Stopwatch 的正確用法"
date: 2024-11-22T00:30:00+08:00
lastmod: 2024-11-22T00:30:31+08:00
draft: false
tags: ["dotnet","csharp","benchmark"]
slug: "stopwatch-best-practice"
---

## Stopwatch 的正確用法

前幾天從 Cash 大的粉專上看到 [Cash Wu Geek](https://www.facebook.com/share/p/1AgUkr1iaC/) 分享了 youtuber-Nick Chapsas 對於 `Stopwatch` 的正確用法：[Youtube:How to Measure Time Correctly in .NET](https://www.youtube.com/watch?v=Lvdyi5DWNm4)，覺得值得仔細了解一下，快速筆記一下心得。

.NET 7 開始提供了 `Stopwatch` 的新 api：`GetTimestamp` 透過 `Diagnostics` 命名空間提供的，可以幫助我們更容易地使用 `Stopwatch` 來計算程式碼的執行時間。

## 基本環境說明

- macOS Sequoia 15.0.1 (Apple M2 Pro)
- dotnet sdk 7.0.410
- dotnet sdk 8.0.401
- dotnet sdk 9.0.100
- JetBrains Rider 2024.3
- NuGet package

    - BenchmarkDotNet 0.14.0

## 使用方式

- .NET 6 之前的用法

    ```csharp
    Stopwatch stopwatch = Stopwatch.StartNew();
    // 執行程式碼
    Console.WriteLine($"Elapsed time: {stopwatch.Elapsed}");
    ```

- .NET 7 新增的 api：`GetTimestamp`

    > 1. 透過 `Stopwatch.GetTimestamp()` 取得系統時間
    > 2. 使用 `Stopwatch.GetElapsedTime(long timestamp)` 取得執行時間

    ```csharp
    long timestamp = Stopwatch.GetTimestamp();
    // 執行程式碼
    Console.WriteLine($"Elapsed time: {Stopwatch.GetElapsedTime(timestamp)}");
    ```

## 心得

- 新語法的優點：

    1. 避免建立新物件而產生的記憶體分配

        > `new Stopwatch()` 與 `Stopwatch.StartNew()` 都會額外建立新的 Stopwatch 物件而產生記憶體分配，而這個動作僅是為了衡量程式碼的執行時間，這樣的記憶體分配是不必要的，而新的 API 可以避免這個問題
    2. 跨平台優化取得系統時間

        > `Stopwatch.GetTimestamp()` 可以取得系統時間，而這個時間是跨平台的，這樣可以避免不同平台的時間取得方式不同而造成的問題
    3. 隨著 `GetTimestamp` 新增的 `GetElapsedTime` 則提供了簡化處理 `Stopwatch.Frequency` 和 `TimeSpan.TicksPerSecond` 的公式

- 新舊語法在不同版本的 .NET SDK 效能比較

    - 測試程式碼

        {{<gist yowko cce321b875ca44b9ee85e145e7bb0395 "StopwatchBenchmark.cs">}}

    - 測試結果

        ![4allsdk](https://github.com/user-attachments/assets/e366f06a-4e62-436a-96d1-e531936124b3)

    - 個別 sdk 的測試結果

        1. .NET 7

            ![1net7](https://github.com/user-attachments/assets/71112630-eef7-41d0-af5b-4baccc6533c1)

        2. .NET 8

            ![2net8](https://github.com/user-attachments/assets/4788e329-1810-4595-99ed-194d216db72a)

        3. .NET 9

            ![3net9](https://github.com/user-attachments/assets/ce2b27ef-6d2b-4673-bb10-47666ea51b8f)

    可以看到在 .NET 7 與 .NET 8 上 `Stopwatch.StartNew` 確實存在 memory allocated 的問題，執行速度也存在差異，但 .NET 9 就沒看到 `Stopwatch.StartNew` memory allocated 與執行速度差異的現象，我粗略查了一下官方文件沒有發現有針對 Stopwatch 效能提升的介紹說明，看了 IL code 新舊語法也確實不同，我再找時間翻翻 source code，有找到再補充。

## 參考資訊

1. [Cash Wu Geek](https://www.facebook.com/share/p/1AgUkr1iaC/)
2. [Microsoft:Performance Improvements in .NET 7](https://devblogs.microsoft.com/dotnet/performance_improvements_in_net_7/?WT.mc_id=DOP-MVP-5002594#diagnostics)
3. [dotnet runtime issue:Add Stopwatch.GetElapsedTime](https://github.com/dotnet/runtime/pull/66372?WT.mc_id=DOP-MVP-5002594)
4. [dotnet runtime issue:API Proposal: Stopwatch.GetElapsedTime](https://github.com/dotnet/runtime/issues/65858?WT.mc_id=DOP-MVP-5002594)
5. [Youtube:How to Measure Time Correctly in .NET](https://www.youtube.com/watch?v=Lvdyi5DWNm4)
6. [Youtube:Are you using the Stopwatch efficiently in .NET?](https://www.youtube.com/watch?v=NTz99yN2urc)
