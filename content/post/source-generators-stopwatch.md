---
title: "使用 Source Generators 來為 method 加上時間測量"
date: 2024-12-03T00:30:00+08:00
lastmod: 2024-12-03T00:30:31+08:00
draft: false
tags: ["dotnet","csharp","benchmark"]
slug: "source-generators-stopwatch"
---

## 使用 Source Generators 來為 method 加上時間測量

之前筆記 [Stopwatch 的正確用法](/stopwatch-best-practice) 紀錄了使用在 .NET 7 所發表 Stopwatch 的新 api 來測量程式執行時間以避免 memory allocate，接著就想起 MethodTimer.Fody：[使用 MethodTimer.Fody 來為 method 加上時間測量](/measure-method-performance-with-methodtimer-fody) 使用的是 `Stopwatch.StartNew()` 雖然 MethodTimer.Fody 有建立 branch 準備套用 Stopwatch 新 API，但一直沒有實際 merge，所以這次嘗試使用 Source Generators 來自動加上 Stopwatch 測量執行時間的程式碼。

以下比較 Fody 與 Source Generators 內容由 chatgpt 提供：

---

Fody 和 Source Generators 都是用來增強 C# 程式開發的工具，但它們運作的方式、應用場景和設計理念存在顯著差異。以下是詳細的比較：

1. 工作時機

   - Fody：
       - 工作階段：Fody 是一個編譯後工具（Post-Compiler Tool）。它修改的是已生成的 IL（中間語言），也就是在 .NET 程式集已經生成後進行操作。
       - 影響範圍：直接修改輸出的程序集（DLL 或 EXE）。

   - Source Generators：
       - 工作階段：Source Generators 是 Roslyn 編譯器的一部分，運行在編譯期間，生成新的源代碼文件供編譯器使用。
       - 影響範圍：生成新的 C# 源代碼，與原始代碼一同編譯。

2. 原理與運作方式

   - Fody：
       - IL 操作：通過修改中間語言（IL），可以直接改變程式的執行行為或增加功能。
       - 無法生成新的代碼文件：因為它是在 IL 層級工作，對開發者不可見。
       - 基於插件的架構：使用「紡織機（Weavers）」來完成具體的功能（如 `PropertyChanged.Fody`）。

   - Source Generators：
       - 生成新代碼：會創建新的 C# 源文件，這些文件對開發者可見（通常出現在 Visual Studio 的 "Generated Code" 中）。
       - 透明性更高：開發者可以看到生成的代碼，便於調試和理解。
       - Roslyn API：依賴 Roslyn 提供的代碼分析和生成功能。

3. 開發體驗

   - Fody：
       - 無侵入性：不需要直接修改源代碼，功能在編譯後自動應用。
       - 隱藏代碼細節：開發者看不到插入的代碼，這可能會在調試時增加難度。
       - 需要學習 IL：對於自定義 Weavers，可能需要了解中間語言。

   - Source Generators：
       - 可見代碼：開發者可以清楚地看到生成的代碼，容易排查問題。
       - 靈活性更高：可以根據需求生成任意形式的 C# 代碼，而不僅限於修改已有代碼。
       - 完全基於 C#：無需接觸 IL，開發門檻更低。

4. 性能
   - Fody：
       - 潛在性能損耗：修改 IL 可能導致一些額外的執行時性能開銷。
       - 編譯時間影響小：因為它在編譯後工作，不會影響原始代碼的編譯速度。

   - Source Generators：
       - 高效執行：生成的代碼在編譯前插入，因此沒有額外的執行時開銷。
       - 增加編譯時間：在大規模項目中，可能會因為代碼生成步驟導致稍微更長的編譯時間。

5. 使用場景

    | 需求 | Fody | Source Generators |
    |:--|:---|:--------|
    | 修改現有代碼行為 | ✅（IL 編輯） | ❌（無法直接修改現有代碼）  |
    | 生成新代碼 | ❌ | ✅ |
    | 自動化 INotifyPropertyChanged 實現 | ✅（PropertyChanged.Fody）| ✅（例如使用 Source Generators 自行實現） |
    | 嵌入 DLL（如嵌入資源文件） | ✅（Costura.Fody） | ❌（不適合這類場景）        |
    | 代碼可見性（生成後的代碼透明度）| ❌（代碼不可見） | ✅（生成的代碼清晰可見）    |
    | 與工具鏈整合（如 IDE 支援）| 一般，需手動配置 | 很好，原生支持 Visual Studio 和 MSBuild    |

6. 哪個更適合你的需求？
   - 使用 Fody 如果：
     - 需要修改現有代碼的行為（如添加方法執行日誌或參數檢查）。
     - 想避免手動生成代碼，而更傾向於在編譯後自動應用功能。
     - 面對的功能需求與現有 Weavers 插件相符。

   - 使用 Source Generators 如果：
     - 希望生成新的代碼來簡化開發過程（如自動生成數據模型或接口）。
     - 需要清晰的生成代碼來進行調試和維護。
     - 更希望直接操作 C# 代碼而非接觸 IL。

7. 結論
Fody 更像是 AOP 工具，適合通用功能的自動化處理；而 Source Generators 更像是一種編譯期代碼生成工具，提供更靈活的代碼生成能力。選擇哪一種技術取決於具體需求和項目架構。

---
以上比較 Fody 與 Source Generators 內容由 chatgpt 提供

## 基本環境說明

- macOS Sequoia 15.1.1 (Apple M2 Pro)
- dotnet sdk 9.0.100
- JetBrains Rider 2024.3
- NuGet package

    - Microsoft.CodeAnalysis.Analyzers 3.11.0
    - Microsoft.CodeAnalysis.Analyzers 3.3.4
    - Microsoft.CodeAnalysis.CSharp 4.11.0
- Console project：SourceGeneratorsDemo StopwatchBenchmark.cs

    {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "StopwatchBenchmark.cs">}}

## 設定方式

- Source Generators 專案：`SourceGenerators_ns20`

    1. 建立 `netstandard2.0` 或是 `netstandard2.1` class library 專案

        設定為 `netstandard2.0` 或是 `netstandard2.1` 可以讓產生出的程式碼出現在 Rider solution explorer 的 project 的 Dependencies 中，雖然設定為 `net90` 也可以正常執行 但產出結果無法出現在 Dependencies 中

        ![1dependencies](https://github.com/user-attachments/assets/a05fa508-3e8f-4a08-8425-9b673c27d044)

        ![3net9runok](https://github.com/user-attachments/assets/9c0221e2-13e6-41ac-a5dd-9720fa9a1e67)

    2. 加入 nuget package

        ```bash
        dotnet add package Microsoft.CodeAnalysis.CSharp --version 4.11.0
        ```

        Microsoft.CodeAnalysis.CSharp 4.11.0 相依於 Microsoft.CodeAnalysis.Analyzers 3.3.4 以上版本，最新版 Microsoft.CodeAnalysis.Analyzers 3.11.0，也是 Microsoft.CodeAnalysis.Analyzers 3.3.4 之後唯一非 beta 版本，但這版開始正式將 Non-incremental source generators 禁用，如果還是想用 Source Generators 的寫法就要避免使用 Microsoft.CodeAnalysis.Analyzers 3.11.0 以上版本

        - 錯誤訊息

            {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "err.txt">}}

        - 錯誤截圖

            ![2err](https://github.com/user-attachments/assets/fc405203-69a4-4b3d-9dd5-31d29d4bd02c)

    3. 新增 `TimedAttribute`

        > 用來標記要測量時間的 method，讓 source generator 可以正確取得

        {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "TimedAttribute.cs">}}

    4. 新增 `TimingGeneratorExecute` class 套用 Generator attribute 並實作 ISourceGenerator

        我不確定兩者實際差異，根據 [GitHub:Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md?WT.mc_id=DOP-MVP-5002594) 節錄兩者差異並將標題拿來用，依我設定的情境兩者皆可以正常運行

        - Generated class (單純實作 `Execute`)

            > 可以在 build 時產生一個全新 class 提供使用者在程式碼中引用

            {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "TimingGeneratorExecute.cs">}}

        - Augment user code (使用 `SyntaxReceiver`)

            > 強化既有的 class，將現有的 class 設為 partial class 並透過 attribute 或是 name 標記特定的 class，讓 SyntaxReceiver 可以在 build 時產生提供額外功能的 partial class

            {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "TimingGeneratorReceiver.cs">}}

    5. 修改 project file

        > source generator 需要設定 `EnforceExtendedAnalyzerRules` 為 `true`，這樣才能正確執行

        ![4projenforce](https://github.com/user-attachments/assets/2454199c-250e-422a-9594-d5b6d19ca930)

        ```xml
            <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
        ```

        {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "SourceGenerators_ns20.csproj">}}

- 實際執行專案：`SourceGeneratorsDemo`

    1. 將 Source Generators 專案加入參考並設定參考類型

        `OutputItemType="Analyzer" ReferenceOutputAssembly="false"` 是常見的設定方式

        - `OutputItemType="Analyzer"`：設定為 Analyzer 類型，這樣才能正確執行
        - `ReferenceOutputAssembly="false"`：不需要將生成的 class 加入參考

        但我選擇將 `TimedAttribute` 搬至 Source Generators 專案中，所以調整 `ReferenceOutputAssembly="true"`，這樣才能正確取得 `TimedAttribute`

        {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "SourceGeneratorsDemo.csproj">}}

    2. 將 `StopwatchBenchmark` class 改為 `partial` 並將 StopwatchBenchmark 的 method 加上 `Timed` attribute

        > 設定 attribute 的 class，不能使用 file-scoped namespace，否則會讓 source generator 無法正確取得 namespace

        {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "StopwatchBenchmarkAfter.cs">}}

    3. 實際使用

        {{<gist yowko 465304a32be5ff6c1fface231b0e17cb "Program.cs">}}

## 心得

1. Source Generators 專案 framework 必需要是 `netstandard2.0` 或是 `netstandard2.1`

    這個我傾向是 IDE 顯示的問題，雖然用 `net9.0` 可以正常執行，但 IDE 會出現解析不到目標方法的提示，所以我傾向使用 `netstandard2.0` 或是 `netstandard2.1`，加上在 Dependencies 中可以看到生成的 class，更能確保產出的結果符合預期

2. Incremental Generators 取代了 Source Generators

    [GitHub:Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md?WT.mc_id=DOP-MVP-5002594) 與 [GitHub:Source Generators](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.md?WT.mc_id=DOP-MVP-5002594) 都有提到 Incremental Generators 取代了 Source Generators

    Incremental Generators 的使用可以參考筆記 [使用 Incremental Generators 來為 method 加上時間測量](/incremental-generators-stopwatch)

3. 新版本 Microsoft.CodeAnalysis.Analyzers 3.11.0 會出現無法編譯

    ![2err](https://github.com/user-attachments/assets/fc405203-69a4-4b3d-9dd5-31d29d4bd02c)

4. debug 困難

    我自己的經驗有幾個：
    1. 沒有產生出 class，但不知道卡在哪一步：標記錯誤？型別不對？還是 source 結構異常？方法用錯？
    2. Source Generators 的 class 都不是常見的，不知道哪些 property 可以使用

    如果使用的是 JetBrains Rider 可以參考筆記 []()

5. 沒有正式文件？

    這個不知道是我沒找到還是真的沒有，我找到的都是 GitHub 上的文件，Microsoft Learn 沒有找到相關的教學，這點有點可惜

完整程式碼可以參考 [GitHub:yowko/source-generators-stopwatch](https://github.com/yowko/source-generators-stopwatch)

## 參考資訊

1. [Stopwatch 的正確用法](/stopwatch-best-practice)
2. [使用 MethodTimer.Fody 來為 method 加上時間測量](/measure-method-performance-with-methodtimer-fody)
3. [.NET Blog:Introducing C# Source Generators](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/?WT.mc_id=DOP-MVP-5002594)
4. [.NET Blog:New C# Source Generator Samples](https://devblogs.microsoft.com/dotnet/new-c-source-generator-samples/?WT.mc_id=DOP-MVP-5002594)
5. [GitHub:Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md?WT.mc_id=DOP-MVP-5002594)
6. [GitHub:Source Generators](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.md?WT.mc_id=DOP-MVP-5002594)
7. [GitHub:samples/CSharp/SourceGenerators](https://github.com/dotnet/roslyn-sdk/tree/main/samples/CSharp/SourceGenerators)
8. [Converting between types in increasingly absurd ways](https://medium.com/@sunside/converting-between-types-in-increasingly-absurd-ways-89414ae6eb7c)
9. [使用 Incremental Generators 來為 method 加上時間測量](/incremental-generators-stopwatch)
10. []()
11. [GitHub:yowko/source-generators-stopwatch](https://github.com/yowko/source-generators-stopwatch)
