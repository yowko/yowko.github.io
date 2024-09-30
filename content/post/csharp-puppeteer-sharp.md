---
title: "使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容"
date: 2024-09-23T00:30:00+08:00
lastmod: 2024-09-23T00:30:31+08:00
draft: false
tags: ["csharp","crawler"]
slug: "csharp-puppeteer-sharp"
---

## 使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容

最近團隊有個需求，需要從網頁上抓取一些資料，雖然普遍對於網路爬蟲的第一印象都是 python，但因為團隊中多數成員都是 C# developer，為了非常態性需求改使用 python，不僅開發效率變差，加上後續維護也會因為熟悉度不夠而多花不少時間在重新上手，因此還是決定以 C# 為主，只是爬網頁資料有不少套件可以利用，便想趁這個機會比較一下各家 library 的差異，以下是我比較的幾個 library 與筆記連結：

- HtmlAgilityPack：[使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
- ScrapySharp：[使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
- AngleSharp：[使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
- Puppeteer Sharp：[使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
- Selenium：[使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
- Playwright：[使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
- [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)

Puppeteer Sharp 是 Node.JS Puppeteer API 的 .NET 移植。

Puppeteer Sharp 提供 API 通過 DevTools 協定或 WebDriver BiDi 控制 Chrome 或 Firefox。默認情況下，Puppeteer 在不可見 UI (headless) 中運行

## 基本環境說明

- macOS Sequoia 15.0 (Apple M2 Pro)
- .NET SDK 8.0.401
- NuGet Libraries
    - PuppeteerSharp 20.0.2

## 使用方式

{{<gist yowko 822eef455a50d7691b55e5841192b038 "Program.cs" >}}

## 心得

- 語法原則上就是 JavaScript，只是透過 C# 來執行，少了語法提示跟檢查的功能
- 只能使用標準寫法 `li[class='List(n)']`，不允許 `li.List(n)`

    - 錯誤訊息

        {{<gist yowko 822eef455a50d7691b55e5841192b038 "brackets.err" >}}

    - 錯誤截圖

        ![1error](https://github.com/user-attachments/assets/94b53883-7baa-4e14-871c-70521629ee99)

- 需要先安裝 Chrome 或 Firefox 瀏覽器而造成執行較慢

完整程式碼請參考 [GitHub - yowko/puppeteer-sharp-demo](https://github.com/yowko/puppeteer-sharp-demo)

## 參考資訊

1. [使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
2. [使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
3. [使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
4. [使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
5. [使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
6. [使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
7. [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)
8. [GitHub - puppeteer-sharp](https://github.com/hardkoded/puppeteer-sharp)
9. [Puppeteer Sharp Api Documentation](https://www.puppeteersharp.com/)
10. [7 Best C# Web Scraping Libraries in 2024](https://www.zenrows.com/blog/c-sharp-web-scraping-library#best-c-web-scraping-libraries)
11. [Popular Scraping libraries in C#](https://www.codementor.io/@riza/popular-scraping-libraries-in-c-23u9pjwfc1)
12. [GitHub - yowko/puppeteer-sharp-demo](https://github.com/yowko/puppeteer-sharp-demo)
