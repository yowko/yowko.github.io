---
title: "使用 C# 搭配 ScrapySharp 來獲取網頁內容"
date: 2024-09-21T00:30:00+08:00
lastmod: 2024-09-21T00:30:31+08:00
draft: false
tags: ["csharp","crawler"]
slug: "csharp-scrapysharp"
---

## 使用 C# 搭配 ScrapySharp 來獲取網頁內容

最近團隊有個需求，需要從網頁上抓取一些資料，雖然普遍對於網路爬蟲的第一印象都是 python，但因為團隊中多數成員都是 C# developer，為了非常態性需求改使用 python，不僅開發效率變差，加上後續維護也會因為熟悉度不夠而多花不少時間在重新上手，因此還是決定以 C# 為主，只是爬網頁資料有不少套件可以利用，便想趁這個機會比較一下各家 library 的差異，以下是我比較的幾個 library 與筆記連結：

- HtmlAgilityPack：[使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
- ScrapySharp：[使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
- AngleSharp：[使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
- Puppeteer Sharp：[使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
- Selenium：[使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
- Playwright：[使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
- [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)

ScrapySharp 的特色

- 依賴 HtmlAgilityPack
- 同時支援 CSS Selector 與 XPath
- 有Web 用戶端，能夠模擬真實的 Web 瀏覽器（處理 referrer、cookie ... 等）

## 基本環境說明

- macOS Sequoia 15.0 (Apple M2 Pro)
- .NET SDK 8.0.401
- NuGet Libraries
    - ScrapySharp 3.0.0

## 使用方式

{{<gist yowko f0f7867ab2882078d8fe0b708a19b76e "Program.cs">}}

## 心得

- 方式名稱沒有按照 JavaScript 的標準 api，好處是避免淆，缺點就是不太直覺
- 只能使用標準寫法 `li[class='List(n)']`，不允許 `li.List(n)`

    - 錯誤訊息

        {{<gist yowko f0f7867ab2882078d8fe0b708a19b76e "bracket.err">}}

    - 錯誤截圖

        ![1error](https://github.com/user-attachments/assets/9d21c812-f7cb-4a90-9fa2-59db7c8c0a02)

- 不允許多個 selector

    - 錯誤訊息

        {{<gist yowko f0f7867ab2882078d8fe0b708a19b76e "mutipleseletor.err">}}

    - 錯誤截圖

        ![2error](https://github.com/user-attachments/assets/39f77aeb-5dea-4f9b-994f-b71d9402cd71)

完整程式碼請參考 [GitHub - yowko/scrapy-harp-demo](https://github.com/yowko/scrapy-harp-demo)

## 參考資訊

1. [使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
2. [使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
3. [使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
4. [使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
5. [使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
6. [使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
7. [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)
8. [GitHub - ScrapySharp](https://github.com/rflechner/ScrapySharp)
9. [7 Best C# Web Scraping Libraries in 2024](https://www.zenrows.com/blog/c-sharp-web-scraping-library#best-c-web-scraping-libraries)
10. [Popular Scraping libraries in C#](https://www.codementor.io/@riza/popular-scraping-libraries-in-c-23u9pjwfc1)
11. [ScrapySharp: Comprehensive Tutorial for Scrapy in C# [2024]](https://www.zenrows.com/blog/scrapysharp#why-you-should-use-scrapysharp)
12. [GitHub - yowko/scrapy-harp-demo](https://github.com/yowko/scrapy-harp-demo)
