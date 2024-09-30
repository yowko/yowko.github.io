---
title: "C# Crawler 套件 Benchmark"
date: 2024-09-27T00:30:00+08:00
lastmod: 2024-09-27T00:30:31+08:00
draft: false
tags: ["csharp","crawler","benchmark"]
slug: "csharp-crawler-benchmark"
---

## C# Crawler 套件 Benchmark

最近團隊有個需求，需要從網頁上抓取一些資料，雖然普遍對於網路爬蟲的第一印象都是 python，但因為團隊中多數成員都是 C# developer，為了非常態性需求改使用 python，不僅開發效率變差，加上後續維護也會因為熟悉度不夠而多花不少時間在重新上手，因此還是決定以 C# 為主，只是爬網頁資料有不少套件可以利用，便想趁這個機會比較一下各家 library 的差異，以下是我比較的幾個 library 與筆記連結：

- HtmlAgilityPack：[使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
- ScrapySharp：[使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
- AngleSharp：[使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
- Puppeteer Sharp：[使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
- Selenium：[使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
- Playwright：[使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
- [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)

紀錄完各家 library 的使用方式後，接下來就是要進行效能測試，這次的測試主要是比較各家 library 在爬取網頁資料時的效能，因此我們會使用同一個網頁來進行測試，並且比較各家 library 在爬取網頁資料時的效能

## 基本環境說明

- macOS Sequoia 15.0 (Apple M2 Pro)
- .NET SDK 8.0.401
- NuGet Libraries
    - BenchmarkDotNet 0.14.0
    - HtmlAgilityPack 1.11.66
    - HtmlAgilityPack.CssSelectors.NetCore 1.2.1
    - Fizzler.Systems.HtmlAgilityPack 1.2.1
    - ScrapySharp 3.0.0
    - AngleSharp 1.1.2
    - AngleSharp.XPath 2.0.4
    - PuppeteerSharp 20.0.2
    - Selenium.WebDriver 4.25.0
    - Selenium.WebDriver.ChromeDriver 129.0.6668.7000
    - Microsoft.Playwright 1.47.0

## 測試程式碼

{{<gist yowko 5d820b37025499467ea0cbb474b19a33 "Program.cs">}}

## 心得

1. 指定 Job ：`SimpleJob(RunStrategy.Monitoring, launchCount: 2, warmupCount: 2, iterationCount: 20)`

    > 其中測試的變數：`UseHtmlString` 表示是否透過 HttpClient 取得 html 內容後再手動設定給 library 進行解析，如果為 `false` 則會直接透過 library 取得網頁內容進行解析，`UseHtmlString` 為 `true` 時，主要是比較各家 library 在解析 html 時的效能，`UseHtmlString` 為 `false` 時 就是比較各家 library 的實際效能，因此理論上應該只看 `UseHtmlString` 為 `false` 的結果

    這不是真正的 Benchmark，而是一個簡單的測試，我們只會執行 20 次，並且在每次執行前都會先執行 2 次 warmup 以確保 JIT 編譯完成，因為不指定的話，在 PuppeteerSharp、Selenium、Playwright 這三個 library 會因為啟動瀏覽器造成資源不足或是效能不穩定，以至於無法正常執行測試

    ![1simplejobboth](https://github.com/user-attachments/assets/c31e20f7-cf24-48d3-b944-48e9a898cf38)

2. 指定 Job ：`SimpleJob(RunStrategy.Monitoring, launchCount: 2, warmupCount: 2, iterationCount: 20)`

    這次測試的變數：`UseHtmlString` 為 `false`

    ![3simplejobfalse](https://github.com/user-attachments/assets/bab49f0c-702b-479d-ad1c-d46ec02b0193)

3. htmlAgilityPack、ScrapySharp、AngleSharp 完整測試

    單純比較不啟動瀏覽器的 library，因此不會有 PuppeteerSharp、Selenium、Playwright 的測試結果，但可能僅能適用於靜態網頁，無法處理動態網頁

    ![2nobrowser](https://github.com/user-attachments/assets/ffdf888a-3d29-4f57-8be4-0feea0a9a869)

測試過程中，陸陸續續遇到 timeout 問題或是執行錯誤，所以程式碼中可以看到一些放寬 timeout 或是瀏覽器相關設定，這些都是為了確保測試能夠正常執行，不過在實際使用上，倒是沒遇過問題，至少設定 20 次的測試都能正常執行

完整程式碼請參考 [GitHub - yowko/csharp-crawler-benchmark](https://github.com/yowko/csharp-crawler-benchmark)

## 參考資料

1. [使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
2. [使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
3. [使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
4. [使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
5. [使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
6. [使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
7. [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)
8. [BenchmarkDotNet](https://benchmarkdotnet.org/index.html)
9. [SOLVED! HTTP request to the remote WebDriver server for URL http://localhost:52847/….. timed out after 60 seconds.](https://seleniumjava.com/2019/09/02/solved-the-http-request-to-the-remote-webdriver-server-for-url-http-localhost52847-session-value-timed-out-after-60-seconds/)
10. [lib/PuppeteerSharp/Helpers/TaskHelper.cs](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Helpers/TaskHelper.cs/)
11. [GitHub - yowko/csharp-crawler-benchmark](https://github.com/yowko/csharp-crawler-benchmark)
