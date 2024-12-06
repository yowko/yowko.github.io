---
title: "使用 Incremental Generators 來為 method 加上時間測量"
date: 2024-12-05T00:30:00+08:00
lastmod: 2024-12-05T00:30:31+08:00
draft: false
tags: ["dotnet","csharp","benchmark"]
slug: "incremental-generators-stopwatch"
---

## 使用 Incremental Generators 來為 method 加上時間測量

之前筆記 [Stopwatch 的正確用法](/stopwatch-best-practice) 紀錄了使用在 .NET 7 所發表 Stopwatch 的新 api 來測量程式執行時間以避免 memory allocate，接著就想起 MethodTimer.Fody：[使用 MethodTimer.Fody 來為 method 加上時間測量](/measure-method-performance-with-methodtimer-fody) 使用的是 `Stopwatch.StartNew()` 雖然 MethodTimer.Fody 有建立 branch 準備套用 Stopwatch 新 API，但一直沒有實際 merge，筆記 [使用 Source Generators 來為 method 加上時間測量](/source-generators-stopwatch) 紀錄使用 Source Generators 來自動加上 Stopwatch 測量執行時間的程式碼。但

以下比較 **Source Generators** 和 **Incremental Generators** 內容由 chatgpt 提供：

---

在 C# 中，**Source Generators** 和 **Incremental Generators** 都屬於 **Roslyn 編譯器** 的功能，用於在編譯時生成代碼。但 Incremental Generators 是對傳統 Source Generators 的改進版本，解決了一些性能和可用性問題。以下是它們的主要區別與比較：

---

1. 基本概念

    - Source Generators：
        - **標準生成器**，於 .NET 5 中引入。
        - 在每次編譯時都會執行完整的代碼生成過程。
        - 不考慮輸入數據的改變範圍，每次都從頭開始處理。

    - Incremental Generators**：
        - 是 .NET 6 中新增的特性，作為 Source Generators 的增強版本。
        - **增量式處理**：只處理受影響的輸入，減少重複工作。
        - 更注重性能和編譯效率，適合大規模代碼生成場景。

2. 工作原理

    - Source Generators：
        - 每次執行時，會從編譯器中獲取所有相關輸入（例如語法樹、符號等），進行完整處理。
        - 沒有內建的增量機制，即使輸入只有部分改變，整個生成器仍需重新執行。

        - 示例：

            ```csharp
            [Generator]
            public class MyGenerator : ISourceGenerator
            {
                public void Execute(GeneratorExecutionContext context)
                {
                    // 獲取所有語法樹，進行處理
                    foreach (var syntaxTree in context.Compilation.SyntaxTrees)
                    {
                        // 生成代碼
                    }
                }

                public void Initialize(GeneratorInitializationContext context) { }
            }
            ```

    - Incremental Generators：
        - 基於新的 `IncrementalGenerator` 接口，通過 **增量執行管道** 處理輸入。
        - 編譯器會追踪輸入的改變，僅重新執行受影響的部分，而非整個生成器。
        - 支援更高效的輸入-輸出數據流管理。
        - 範例

            ```csharp
            [Generator]
            public class MyIncrementalGenerator : IIncrementalGenerator
            {
                public void Initialize(IncrementalGeneratorInitializationContext context)
                {
                    // 定義增量管道
                    var syntaxProvider = context.SyntaxProvider
                        .CreateSyntaxProvider(
                            predicate: (node, _) => node is ClassDeclarationSyntax,
                            transform: (context, _) => (ClassDeclarationSyntax)context.Node
                        )
                        .Collect();

                    // 將處理後的結果用於生成代碼
                    context.RegisterSourceOutput(syntaxProvider, (context, classes) =>
                    {
                        foreach (var classSyntax in classes)
                        {
                            context.AddSource(
                                $"{classSyntax.Identifier.Text}.Generated.cs",
                                $"// Auto-generated class: {classSyntax.Identifier.Text}"
                            );
                        }
                    });
                }
            }
            ```

3. 性能比較

    - Source Generators：
        - **非增量**：每次都重新處理所有輸入，無法跳過未改變的部分。
        - 適合小型項目或簡單的代碼生成邏輯。
        - 在大型項目中可能導致 **不必要的計算開銷**。

    - Incremental Generators：
        - **增量處理**：只重新處理輸入有變化的部分，避免重複工作。
        - 更高效，適合處理大量輸入或多步驟的生成邏輯。
        - **數據流模型**：將輸入分成小單元，按需處理，減少內存和 CPU 使用。

4. 開發體驗

    | **功能**                  | **Source Generators**              | **Incremental Generators**            |
    |-----------|-------------------------------------|---------------------------------------|
    | **性能**                  | 每次執行完整處理，效率較低          | 增量執行，僅處理受影響部分，性能更高   |
    | **用法**                  | 簡單直接，適合基本的代碼生成需求    | 較複雜，但適合高效、可擴展的場景      |
    | **輸入管理**              | 手動篩選和處理所有輸入              | 自動增量追踪輸入變化                  |
    | **代碼可讀性**            | 更直接，但在複雜場景中易失控        | 結構化的數據流模型，適合大型項目      |

5. 適用場景
    - Source Generators：
        - 小型項目或一次性代碼生成。
        - 對性能要求不高的場景。
        - 開發者想快速上手並構建基本生成器功能。

    - Incremental Generators：
        - 大型項目，尤其是編譯輸入（如語法樹、符號）變化頻繁的場景。
        - 需要高效的增量式處理，減少無效計算。
        - 對性能和內存使用有更高要求的應用。

6. 總結
    - 如果你只是嘗試編寫簡單的代碼生成邏輯，**Source Generators** 已經能滿足需求。
    - 如果你需要處理大量輸入、應用更複雜的邏輯，或者希望提升性能，**Incremental Generators** 是更好的選擇。
    - 使用 Incremental Generators 雖然學習曲線稍高，但其增量特性可以顯著改善開發體驗和性能，尤其是在大型或持續增長的項目中。

---
以上比較 **Source Generators** 和 **Incremental Generators** 內容由 chatgpt 提供

## 基本環境說明

- macOS Sequoia 15.1.1 (Apple M2 Pro)
- dotnet sdk 9.0.100
- JetBrains Rider 2024.3
- NuGet package

    - Microsoft.CodeAnalysis.CSharp 4.12.0
- Console project：IncrementalGeneratorsDemo StopwatchBenchmark.cs

    {{<gist yowko f98658f1a2c7f5628be7da84f9b63ff6 "StopwatchBenchmark.cs">}}

## 設定方式

- Incremental Generators 專案：`IncrementalGenerators_ns20`

    1. 建立 `netstandard2.0` 或是 `netstandard2.1` class library 專案

        設定為 `netstandard2.0` 或是 `netstandard2.1` 可以讓產生出的程式碼出現在 Rider solution explorer 的 project 的 Dependencies 中，雖然設定為 `net90` 也可以正常執行 但產出結果無法出現在 Dependencies 中

        ![1dependencies](https://github.com/user-attachments/assets/79ddc7f9-9534-4d29-bbd0-f6865a0feeed)

        ![2net9runok](https://github.com/user-attachments/assets/71ae1834-ca1d-43ba-9524-3929543dc3a9)

    2. 加入 nuget package

        ```bash
        dotnet add package Microsoft.CodeAnalysis.CSharp --version 4.11.0
        ```

    3. 新增 `TimerGenerator` class 加上 Generator attribute 並實作 IIncrementalGenerator

        {{<gist yowko f98658f1a2c7f5628be7da84f9b63ff6 "TimerGenerator.cs">}}

    4. 修改 project file

        > source generator 需要設定 `EnforceExtendedAnalyzerRules` 為 `true`，這樣才能正確執行

        ![3projenforce](https://github.com/user-attachments/assets/105bd661-3a74-4ff8-8b41-40b62514606a)

        ```xml
        <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
        ```

        {{<gist yowko f98658f1a2c7f5628be7da84f9b63ff6 "IncrementalGenerators_ns20.csproj">}}

- 實際執行專案：`IncrementalGeneratorsDemo`

    1. 將 Incremental Generators 專案加入參考並設定參考類型

        `OutputItemType="Analyzer" ReferenceOutputAssembly="true"`

        - `OutputItemType="Analyzer"`：設定為 Analyzer 類型，這樣才能正確執行
        - `ReferenceOutputAssembly="true"`：將生成的 class 加入參考

        `OutputItemType="Analyzer" ReferenceOutputAssembly="false"` 是較為常見的設定方式，但我選擇將 `TimedAttribute` 搬至 Source Generators 專案中，所以調整 `ReferenceOutputAssembly="true"`，這樣才能正確取得 `TimedAttribute`

        {{<gist yowko f98658f1a2c7f5628be7da84f9b63ff6 "IncrementalGeneratorsDemo.csproj">}}

    2. 將 `StopwatchBenchmark` class 改為 `partial` 並將 StopwatchBenchmark 的 method 加上 `Timed` attribute

        {{<gist yowko f98658f1a2c7f5628be7da84f9b63ff6 "StopwatchBenchmarkAfter.cs">}}

    3. 實際使用

        {{<gist yowko f98658f1a2c7f5628be7da84f9b63ff6 "Program.cs">}}

## 心得

1. Source Generators 專案 framework 必需要是 `netstandard2.0` 或是 `netstandard2.1`

    這個我傾向是 IDE 顯示的問題，雖然用 `net9.0` 可以正常執行，但 IDE 會出現解析不到目標方法的提示，所以我傾向使用 `netstandard2.0` 或是 `netstandard2.1`，加上在 Dependencies 中可以看到生成的 class，更能確保產出的結果符合預期

2. debug 困難

    我自己的經驗有幾個：
    1. 沒有產生出 class，但不知道卡在哪一步：標記錯誤？型別不對？還是 source 結構異常？方法用錯？
    2. Incremental Generators 的 class 都不是常見的，不知道哪些 property 可以使用

    如果使用的是 JetBrains Rider 可以參考筆記 [使用 JetBrains Rider 來 Debug Source Generators 或 Incremental Generators](/debug-source-generators-incremental-generators)

3. 沒有正式文件？

    這個不知道是我沒找到還是真的沒有，我找到的都是 GitHub 上的文件，Microsoft Learn 沒有找到相關的教學，這點有點可惜

4. 準正式文件語法沒有遵循規範

    我參考 [GitHub:Incremental Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.cookbook.md?WT.mc_id=DOP-MVP-5002594) 的寫法，但其中寫法在 `netstandard2.0(C# 7.3)` 並不支援，甚至是 `netstandard2.1(C# 8.0)` 都不支援，不是說好只能用 `netstandard2.0` 嗎XD

    - static anonymouos function 必需使用 C# 9 以上版本
    - record class 必需使用 C# 10 以上版本

想要了解更多細節，請參考 [Andrew Lock | .NET Escapades:Creating an incremental generator](https://andrewlock.net/creating-a-source-generator-part-1-creating-an-incremental-source-generator/#7-parsing-the-enumdeclarationsyntax-to-create-an-enumtogenerate)

完整程式碼可以參考 [GitHub:yowko/incremental-generators-stopwatch](https://github.com/yowko/incremental-generators-stopwatch)

## 參考資料

1. [Stopwatch 的正確用法](/stopwatch-best-practice)
2. [使用 MethodTimer.Fody 來為 method 加上時間測量](/measure-method-performance-with-methodtimer-fody)
3. [使用 Source Generators 來為 method 加上時間測量](/source-generators-stopwatch)
4. [GitHub:Incremental Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.cookbook.md?WT.mc_id=DOP-MVP-5002594)
5. [GitHub:Incremental Generators](https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.md?WT.mc_id=DOP-MVP-5002594)
6. [Microsoft Learn:C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning?WT.mc_id=DOP-MVP-5002594)
7. [使用 JetBrains Rider 來 Debug Source Generators 或 Incremental Generators](/debug-source-generators-incremental-generators)
8. [Andrew Lock | .NET Escapades:Creating an incremental generator](https://andrewlock.net/creating-a-source-generator-part-1-creating-an-incremental-source-generator/#7-parsing-the-enumdeclarationsyntax-to-create-an-enumtogenerate)
9. [GitHub:yowko/incremental-generators-stopwatch](https://github.com/yowko/incremental-generators-stopwatch)
