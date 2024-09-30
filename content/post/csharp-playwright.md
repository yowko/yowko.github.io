---
title: "使用 C# 搭配 Playwright 來獲取網頁內容"
date: 2024-09-25T00:30:00+08:00
lastmod: 2024-09-25T00:30:31+08:00
draft: false
tags: ["csharp","crawler"]
slug: "csharp-playwright"
---

## 使用 C# 搭配 Playwright 來獲取網頁內容

最近團隊有個需求，需要從網頁上抓取一些資料，雖然普遍對於網路爬蟲的第一印象都是 python，但因為團隊中多數成員都是 C# developer，為了非常態性需求改使用 python，不僅開發效率變差，加上後續維護也會因為熟悉度不夠而多花不少時間在重新上手，因此還是決定以 C# 為主，只是爬網頁資料有不少套件可以利用，便想趁這個機會比較一下各家 library 的差異，以下是我比較的幾個 library 與筆記連結：

- HtmlAgilityPack：[使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
- ScrapySharp：[使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
- AngleSharp：[使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
- Puppeteer Sharp：[使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
- Selenium：[使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
- Playwright：[使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
- [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)

Playwright 是一個用於 Web 測試和自動化的框架。它允許使用單個 API 測試 Chromium、Firefox 和 WebKit。Playwright 旨在實現常青、功能強大、可靠且快速的跨瀏覽器 Web 自動化。

## 基本環境說明

- macOS Sequoia 15.0 (Apple M2 Pro)
- .NET SDK 8.0.401
- NuGet Libraries
    - Microsoft.Playwright 1.47.0

## 使用方式

{{<gist yowko 960db86640be6d3592804164144e5e75 "Program.cs" >}}

## 心得

- Playwright 本質還是自動化測試框架，在爬蟲上的應用語法上還是沒那麼直覺，加上又不像 Selenium 發展時間長，所以網路上資源相對較少
- Playwright 在 element 的處理上比較像是為元素操作而生 (像是點擊、輸入等)，需要取得內容時就會有點遶：像是 css selector 取得的 element 後的型別是 IElementHandle，沒辦法再使用 xpath 取得其他子元件，需要再透過 document.evaluate 才能使用 xpath
- 只能使用標準寫法 `li[class='List(n)']`，不允許 `li.List(n)`

    - 錯誤訊息

        {{<gist yowko 960db86640be6d3592804164144e5e75 "brackets.err" >}}

    - 錯誤截圖

        ![1error](https://github.com/user-attachments/assets/9142f04a-7643-4b48-b1a3-a9c0ecf9a9d8)

- 如何取得 xpath 或是 selector
    - 開啟瀏覽器的開發者工具並選取要取得的元素

        ![2developertool](https://github.com/user-attachments/assets/f1e243b3-8d93-4ddf-9e2e-706c4acc8292)

    - 右鍵選擇 `Copy` --> `Copy XPath` 或是 `Copy selector`

        ![3copy](https://github.com/user-attachments/assets/f947329f-d163-4c22-b89b-bd94c248522d)

完整程式碼請參考 [GitHub - yowko/playwright-demo](https://github.com/yowko/playwright-demo)

## 參考資訊

1. [使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
2. [使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
3. [使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
4. [使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](csharp-puppeteer-sharp)
5. [使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
6. [使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
7. [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)
8. [GitHub - playwright](https://github.com/microsoft/playwright)
9. [Playwright](https://playwright.tw/)
10. [7 Best C# Web Scraping Libraries in 2024](https://www.zenrows.com/blog/c-sharp-web-scraping-library#best-c-web-scraping-libraries)
11. [Popular Scraping libraries in C#](https://www.codementor.io/@riza/popular-scraping-libraries-in-c-23u9pjwfc1)
12. [GitHub - yowko/playwright-demo](https://github.com/yowko/playwright-demo)
