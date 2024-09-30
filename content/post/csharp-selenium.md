---
title: "使用 C# 搭配 Selenium 來獲取網頁內容"
date: 2024-09-24T00:30:00+08:00
lastmod: 2024-09-24T00:30:31+08:00
draft: false
tags: ["csharp","crawler"]
slug: "csharp-selenium"
---

## 使用 C# 搭配 Selenium 來獲取網頁內容

最近團隊有個需求，需要從網頁上抓取一些資料，雖然普遍對於網路爬蟲的第一印象都是 python，但因為團隊中多數成員都是 C# developer，為了非常態性需求改使用 python，不僅開發效率變差，加上後續維護也會因為熟悉度不夠而多花不少時間在重新上手，因此還是決定以 C# 為主，只是爬網頁資料有不少套件可以利用，便想趁這個機會比較一下各家 library 的差異，以下是我比較的幾個 library 與筆記連結：

- HtmlAgilityPack：[使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
- ScrapySharp：[使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
- AngleSharp：[使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
- Puppeteer Sharp：[使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
- Selenium：[使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
- Playwright：[使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
- [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)

Selenium 主要是用來做 web application 的自動化測試，提供 extension 來支援各種瀏覽器，可以透過 WebDriver 來控制瀏覽器，進行各種操作，如點擊、輸入、獲取元素等。

## 基本環境說明

- macOS Sequoia 15.0 (Apple M2 Pro)
- .NET SDK 8.0.401
- NuGet Libraries
    - Selenium.WebDriver 4.25.0
    - Selenium.WebDriver.ChromeDriver 129.0.6668.7000

## 使用方式

{{<gist yowko 6340bb93bd5ea67e61711e07059a938e "Program.cs" >}}

## 心得

- 語法有自己的特色，跟其他 library 原生用來當爬蟲的不同，但又不像 Playwright 那麼不直覺
- 只能使用標準寫法 `li[class='List(n)']`，不允許 `li.List(n)`

    - 錯誤訊息

        {{<gist yowko 6340bb93bd5ea67e61711e07059a938e "brackets.err" >}}

    - 錯誤截圖

        ![1error](https://github.com/user-attachments/assets/a980ce12-af28-4585-b12c-449b89f1a5a7)

- 如何取得 xpath 或是 selector
    - 開啟瀏覽器的開發者工具並選取要取得的元素

        ![2developertool](https://github.com/user-attachments/assets/f1e243b3-8d93-4ddf-9e2e-706c4acc8292)

    - 右鍵選擇 `Copy` --> `Copy XPath` 或是 `Copy selector`

        ![3copy](https://github.com/user-attachments/assets/f947329f-d163-4c22-b89b-bd94c248522d)

完整程式碼請參考 [GitHub - yowko/selenium-demo](https://github.com/yowko/selenium-demo)

## 參考資訊

1. [使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
2. [使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
3. [使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
4. [使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
5. [使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
6. [使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
7. [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)
8. [Selenium](https://www.selenium.dev/)
9. [7 Best C# Web Scraping Libraries in 2024](https://www.zenrows.com/blog/c-sharp-web-scraping-library#best-c-web-scraping-libraries)
10. [Popular Scraping libraries in C#](https://www.codementor.io/@riza/popular-scraping-libraries-in-c-23u9pjwfc1)
11. [GitHub - yowko/selenium-demo](https://github.com/yowko/selenium-demo)
