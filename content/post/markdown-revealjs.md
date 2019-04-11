---
title: "用 Markdown 與 Reveal.js 來製作簡報"
date: 2016-12-02T00:42:34+08:00
lastmod: 2018-09-04T00:42:34+08:00
draft: false
tags: ["套件","Frontend","JavaScript"]
slug: "markdown-revealjs"
aliases:
    - /2016/12/markdown-revealjs.html
---
# 用 Markdown 與 Reveal.js 來製作簡報
相信很多人對於 Markdown 的簡易語法愛不釋手，而 Reveal.js 是一套可以使用 Markdown 語法以 HTML 呈現的簡報框架

透過將 `Markdown` 與 `Reveal.js` 結合就可以讓我們在學習過程中所累積下來的筆記，快速地轉換簡報，大大地減少製作簡報所需的時間與精力

# `Reveal.js`的優點
1. 可以用 html 做簡報
    
    - [Reveal.JS 網站](http://lab.hakim.se/reveal`、`js/#/) 就是透過 `Reveal.js` 製作，大家可以去試試

2. 具垂直簡報的特性
    
    - 以 `PowerPoint` 為例，在簡報中的 `下`、`右` 代表著同一件事，就是下一張投影片，`上`、`左`代表著上一張投影片，而在 `Reveal.js` 中，投影片的序列就不是單一的，`Reveal.js` 有水平及垂直兩個維度的概念

    - 兩個維度讓我們更容易去區隔出不同的簡報章節

3. 不同的檢視角度
    - 按 `esc` 概觀所有簡報，方便搜尋
    - `Alt+ click` 可以放大簡報內容

4. 支援觸控
    
    讓簡報者不一者需要電腦，透過行動裝置也可以進行一場精采的簡報

5. 支援 markdown
    
    可以使用 markdown 語法來建立簡報內容

6. 片段顯示及強調
    
    可以簡單地利用 `class` 與 `html attribute` 來達到效果

7. 快速設定轉場效果
    - 利用 `javascript` 的屬性設定，就可以變化不同的轉場效果
    - 預設有 `None`、`Fade`、`Slide`、`Convex`、`Concave`、`Zoom`

8. 快速設定簡報風格
    - 透過載入不同的 css，就可以改變整個簡報風格
    - 預設有 `Black (default)`、`White`、`League`、`Sky`、`Beige`、`Simple Serif`、` Blood`、`Night`、`Moon`、`Solarized`


# 如何與 `Markdown` 結合
1. clone 專案

    從 [gihub](https://github.com/hakimel/reveal.js) clone 整個專案，然後以 website 啟動專案(ex：iis 站台、webmatrix 以 website 開啟....)

2. 加上 `markdown` 檔案區段

    在根目錄的 `index.html` 中，加入想要透過 Reveal.js 呈現簡報的 markdown 檔案及分隔符號(加在 `body` --> `div[class='reveal']` --> `div[class='slides']` 中)     
    ``` 
	<section data-markdown="intro.md" data-separator="------" data-separator-vertical="----"></section>
    ```

    > ![includemd](https://trello-attachments.s3.amazonaws.com/58019809ff9d696b4443e0f8/1200x606/2c091b8fe4631488ced1c6050a51193b/markdownfile_%E7%BB%93%E6%9E%9C.png)
    > 上圖表示使用網站根目錄下的`intro.md`

    - 讓 Reveal.js 使用 markdown 語法進行簡報主要有兩種方式：

        1. In-html Markdown

            > 將 markdown 寫在 index.html 中 
        2. External Markdown
            
            > 使用外部的 markdown 檔案內容

        因為主要是希望可以透過學習筆記的方式來快速產生簡報，`External Markdown` 才是我們設定的目標
3. `markdown` 設定

    設定分隔方式(官方使用`^n`), 我個人使用上覺得會造成檔案行數太多不好閱讀，所以我改用其他符號，大家也可以依本身喜好自訂分隔符號
    
    - 3-1. `data-separator` 可以指定用來分隔`水平`投影片的符號

        在 markdown 檔案使用 `------`(6個 `-` 符號) 做為分隔

        >![dataseperator](https://trello-attachments.s3.amazonaws.com/58019809ff9d696b4443e0f8/1200x606/f99e17864b7065c2a79b665084deff81/dataseperate_%E7%BB%93%E6%9E%9C.png)

    - 3-2. `data-separator-vertical` 可以指定用來分隔`垂直`投影片的符號

        在 markdown 檔案使用 `----`(4個 `-` 符號) 做為分隔

        >![dataseperatorvertical](https://trello-attachments.s3.amazonaws.com/58019809ff9d696b4443e0f8/1200x606/c89f978514692ce809ab3fdc8cd1afa6/dataseperateverical_%E7%BB%93%E6%9E%9C.png)

# 實際效果
## markdown 檔案內容範例
>![mdfile](https://trello-attachments.s3.amazonaws.com/58019809ff9d696b4443e0f8/580x311/2322dca307303e380ac66f040c0f59df/mdfile_%E7%BB%93%E6%9E%9C.png)

## overview 輸出結果
>![overview](https://trello-attachments.s3.amazonaws.com/58019809ff9d696b4443e0f8/1025x573/48d157e136f47f8907157fdbb1a11839/result_%E7%BB%93%E6%9E%9C.png)



# 心得
## 優點
1. 修改時只要修改一個地方就好(比較符合 OOP 的概念XD)
2. HTML base ，可以自行調整樣式

## 缺點
1. 設定上不像 PowerPoint 可以做很細微的調整
2. 需要建立在網站的基礎上，普遍性不如 PowerPoint 


# 參考資料
1. [Reveal.JS](http://lab.hakim.se/reveal`、`js/#/)
2. [Reveal-js(GITHUB)](https://github.com/hakimel/reveal.js)
3. [Reveal-js(GITHUB)#markdown](https://github.com/hakimel/reveal.js#markdown)