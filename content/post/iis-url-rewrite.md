---
title: "如何透過 IIS 的 URL Rewrite 功能限制存取特定檔案或附檔名"
date: 2017-04-20T01:42:00+08:00
lastmod: 2021-11-02T01:42:00+08:00
draft: false
tags: ["IIS"]
slug: "iis-url-rewrite"
aliases:
    - /2017/04/iis-url-rewrite.html
---
## 如何透過 IIS 的 URL Rewrite 功能限制存取特定檔案或附檔名

之前文章 [如何透過 IIS 的 Request Filtering 功能限制存取特定檔案或附檔名](/iis-request-filtering) 介紹到如何使用 IIS Request Filter Module 來避免特定檔案在網站上裸奔，而同事則是用了 URL Rewrite 一樣達成了目的，順便紀錄一下作法

## 安裝 URL Rewrite

與 Request Filtering 直接使用 Windows 新增移除的安裝方式不同，URL Rewrite 需要另外下載安裝，可以至 [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite) 下載 URL 安裝檔

目前版本(URL Rewrite Module 2.0)可以支援：IIS 7, IIS 7.5, IIS 8, IIS 8.5, IIS 10

安裝方式：

1. 透過 WebPI

    ![1webpi](https://cloud.githubusercontent.com/assets/3851540/25118496/39b55696-2449-11e7-9143-c8dea28741b7.png)

    ![2webpi2](https://cloud.githubusercontent.com/assets/3851540/25118497/39dd15dc-2449-11e7-9f6d-b35bdd4beff6.png)

    ![3webpi3](https://cloud.githubusercontent.com/assets/3851540/25118498/39f17cd4-2449-11e7-8db7-afb495e28982.png)

    ![4webpi4](https://cloud.githubusercontent.com/assets/3851540/25118499/39f36080-2449-11e7-8bbf-7a310ff18c0c.png)

2. 使用 .msi 檔

    ![5msi1](https://cloud.githubusercontent.com/assets/3851540/25118500/39f5a1a6-2449-11e7-9393-86ce6e3b6e7d.png)

    ![6msi2](https://cloud.githubusercontent.com/assets/3851540/25118501/39f988ca-2449-11e7-98dd-0082cd472a10.png)

## 如何設定 URL Rewrite

1. 針對目標站台 --> URL Rewrite

    ![7urlrewrite](https://cloud.githubusercontent.com/assets/3851540/25118502/39fc4d1c-2449-11e7-87ee-8f7a28ee9a0a.png)

2. Add Rule(s)... --> Request Blocking

    ![8addrule](https://cloud.githubusercontent.com/assets/3851540/25118503/3a056dca-2449-11e7-9f2c-7fa86b5d1a1e.png)

3. Block access based on:

    * URL Path (使用 url 來封鎖)

        ![9urlpath](https://cloud.githubusercontent.com/assets/3851540/25118504/3a146456-2449-11e7-804d-2bc5de739c69.png)

4. Block request that:

    * Matches the Pattern (與 pattern 相符時封鎖)

        ![10blockrequest](https://cloud.githubusercontent.com/assets/3851540/25118505/3a155b9a-2449-11e7-933f-87e11b559544.png)

5. Pattern(URL Path):

    ![11urlpattern](https://cloud.githubusercontent.com/assets/3851540/25118506/3a182ee2-2449-11e7-9066-3878f479716a.png)

6. Using

    * Wildcards (使用字元比對)

        ![12using](https://cloud.githubusercontent.com/assets/3851540/25118507/3a1f1252-2449-11e7-86f8-60b271f6f7e2.png)

7. How to Block:

    * Send an HTTP 403 (Forbidden) Response (回傳 403)

        ![13resoinse](https://cloud.githubusercontent.com/assets/3851540/25118508/3a20ea1e-2449-11e7-9b65-4793d07f438d.png)

## 設定完成結果

* 這是依前面設定的內容來呈現的

    ![14result](https://cloud.githubusercontent.com/assets/3851540/25118641/b9c1fe02-2449-11e7-9f3a-ea5b855fc1b4.png)

## 心得

URL Rewrite 感覺使用上也滿方便的，但缺點是需要額外下載安裝，這在某些環境是比較不方便的，當然 URL Rewrite 還有其他功能，其他功能可以參考 [URL Rewrite: 產生具親和力的網址](https://www.microsoft.com/taiwan/technet/iis/expand/URLRewrite.aspx)

## 參考資訊

1. [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
2. [URL Rewrite: 產生具親和力的網址](https://www.microsoft.com/taiwan/technet/iis/expand/URLRewrite.aspx)
3. [如何透過 IIS 的 Request Filtering 功能限制存取特定檔案或附檔名](/iis-request-filtering)
