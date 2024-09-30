---
title: "使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容"
date: 2024-09-20T00:30:00+08:00
lastmod: 2024-09-20T00:30:31+08:00
draft: false
tags: ["csharp","crawler"]
slug: "csharp-htmlagilitypack"
---

## 使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容

最近團隊有個需求，需要從網頁上抓取一些資料，雖然普遍對於網路爬蟲的第一印象都是 python，但因為團隊中多數成員都是 C# developer，為了非常態性需求改使用 python，不僅開發效率變差，加上後續維護也會因為熟悉度不夠而多花不少時間在重新上手，因此還是決定以 C# 為主，只是爬網頁資料有不少套件可以利用，便想趁這個機會比較一下各家 library 的差異，以下是我比較的幾個 library 與筆記連結：

- HtmlAgilityPack：[使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
- ScrapySharp：[使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
- AngleSharp：[使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
- Puppeteer Sharp：[使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
- Selenium：[使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
- Playwright：[使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
- [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)

Html Agility Pack (HAP) 是用 C# 撰寫用於讀/寫 DOM 的 HTML 解析器，支援一般的 XPATH 或 XSLT

## 基本環境說明

- macOS Sequoia 15.0 (Apple M2 Pro)
- .NET SDK 8.0.401
- NuGet Libraries
    - HtmlAgilityPack 1.11.66
    - HtmlAgilityPack.CssSelectors.NetCore 1.2.1
    - Fizzler.Systems.HtmlAgilityPack 1.2.1

## 使用方式

{{<gist yowko 436eee25b9f8d3743495f7262d131ef4 "Program.cs">}}

![1result](https://github.com/user-attachments/assets/f9931820-d739-4e67-817a-59186769933c)

## 心得

- HtmlAgilityPack 原生僅支援 XPath，需要透過 `HtmlAgilityPack.CssSelectors.NetCore` 或是 `Fizzler.Systems.HtmlAgilityPack` 來支援 CSS Selector，雖然 Html Agility Pack 官方推薦 `Fizzler.Systems.HtmlAgilityPack`，但我實際使用起來有幾個不便的地方，我個人比較推薦 `HtmlAgilityPack.CssSelectors.NetCore`

    1. `Fizzler.Systems.HtmlAgilityPack` 是基於 HtmlNode，不如 `HtmlAgilityPack.CssSelectors.NetCore` 直接使用 `HtmlDocument` 便利
    2. `HtmlAgilityPack.CssSelectors.NetCore` 支援 `{html tag}.class` 的寫法，`Fizzler.Systems.HtmlAgilityPack` 則需要使用一般有效的寫法： `{html tag}[class='{class name}']`

- 如何取得 xpath 或是 selector
    - 開啟瀏覽器的開發者工具並選取要取得的元素

        ![2developertool](https://github.com/user-attachments/assets/f1e243b3-8d93-4ddf-9e2e-706c4acc8292)

    - 右鍵選擇 `Copy` --> `Copy XPath` 或是 `Copy selector`

        ![3copy](https://github.com/user-attachments/assets/f947329f-d163-4c22-b89b-bd94c248522d)

完整程式碼請參考 [GitHub - yowko/html-agility-pack-demo](https://github.com/yowko/html-agility-pack-demo)

## 參考資料

1. [使用 C# 搭配 HtmlAgilityPack 來獲取網頁內容](/csharp-htmlagilitypack)
2. [使用 C# 搭配 ScrapySharp 來獲取網頁內容](/csharp-scrapysharp)
3. [使用 C# 搭配 AngleSharp 來獲取網頁內容](/csharp-anglesharp)
4. [使用 C# 搭配 Puppeteer Sharp 來獲取網頁內容](/csharp-puppeteer-sharp)
5. [使用 C# 搭配 Selenium 來獲取網頁內容](/csharp-selenium)
6. [使用 C# 搭配 Playwright 來獲取網頁內容](/csharp-playwright)
7. [C# Crawler 套件 Benchmark](/csharp-crawler-benchmark)
8. [Html Agility Pack (HAP)](https://html-agility-pack.net/)
9. [GitHub - Fizzler.Systems.HtmlAgilityPack](https://github.com/atifaziz/Hazz)
10. [GitHub - HtmlAgilityPack.CssSelectors.NetCore](https://github.com/trenoncourt/HtmlAgilityPack.CssSelectors.NetCore)
11. [7 Best C# Web Scraping Libraries in 2024](https://www.zenrows.com/blog/c-sharp-web-scraping-library#best-c-web-scraping-libraries)
12. [Popular Scraping libraries in C#](https://www.codementor.io/@riza/popular-scraping-libraries-in-c-23u9pjwfc1)
13. [GitHub - yowko/html-agility-pack-demo](https://github.com/yowko/html-agility-pack-demo)
