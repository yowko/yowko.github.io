---
title: "將 Reveal.js 簡報匯出成 PDF"
date: 2016-12-17T00:42:34+08:00
lastmod: 2018-09-06T00:42:34+08:00
draft: false
tags: ["套件","Frontend","JavaScript"]
slug: "revealjs-pdf"
aliases:
    - /2016/12/revealjs-pdf.html
---
# 將 Reveal.js 簡報匯出成 PDF
之前筆記曾經紀錄到如何使用 Reveal.js 搭配 Markdown 語法來製作簡報，可以大幅減少製作簡報的時間成本，但簡報除了實際報告的呈現之外，另一個重點是最後產出的檔案，不少聽眾還是習慣透過簡報檔來回憶簡報的內容，所以今天就來紀錄一下該如何將 Reveal.js 的簡報匯出成 PDF 吧

# 如何匯出？
1. 在簡報 URL 加上`print-pdf` (擇一即可)
    - `http://localhost:9002/?print-pdf` 
    - `http://localhost:9002/index.html?print-pdf#/`

2. 確認頁面有載入特定 css 的 javascript
    - 如果頁面是從 `GITHUB` 專案複製下來修改的可以忽略這段，預設已加入
    
        ```	html
        <!-- Printing and PDF exports -->
        <script>
            var link = document.createElement( 'link' );
            link.rel = 'stylesheet';
            link.type = 'text/css';
            link.href = window.location.search.match( /print-pdf/gi ) ? 'css/print/pdf.css' : 'css/print/paper.css';
            document.getElementsByTagName( 'head' )[0].appendChild( link );
        </script>
        ```
        ![js](https://cloud.githubusercontent.com/assets/3851540/22176557/64db41e0-e048-11e6-8249-3185d24df452.png)

3. 在瀏覽器中按下 `Ctrl+p` 開啟列印視窗

    ![print](https://cloud.githubusercontent.com/assets/3851540/22176549/64b46386-e048-11e6-9013-f5ea24162300.png)


4. 選擇列印`目的地`
    - 4-1. `Chrome` 專屬的 `另存為 PDF`
        - `54.0.2840.71 m (64-bit)` 為例

            ![chrome](https://cloud.githubusercontent.com/assets/3851540/22176547/64b2aa1e-e048-11e6-9155-ef7406c0faa4.png)


        - 勾選`背景圖形` --> 儲存
            
            ![save](https://cloud.githubusercontent.com/assets/3851540/22176552/64b61870-e048-11e6-8d9c-8b7ca889ffe8.png)
        
    - 4-2. Windows 10 的虛擬印表機(Microsoft Print to PDF)
    
        ![windows10](https://cloud.githubusercontent.com/assets/3851540/22176560/64fd317e-e048-11e6-8c31-da8b385baa48.png)

        - 勾選`背景圖形`
            
            ![bg](https://cloud.githubusercontent.com/assets/3851540/22176551/64b5c0b4-e048-11e6-9efe-48c32aabdfae.png)

# 注意事項

>匯出頁面的 URL 加上`print-pdf`後，瀏覽器會出現跑版，但實際產出的 PDF 是正常的

- HTML
    
    ![html](https://cloud.githubusercontent.com/assets/3851540/22176559/64fbc654-e048-11e6-81a1-e80bd1f4185d.png)

- PDF
    
    ![pdf](https://cloud.githubusercontent.com/assets/3851540/22176555/64da5f1e-e048-11e6-9bb8-bedfc26f927a.png)

# 參考資料
1. [Reveal.js(github)](https://github.com/hakimel/reveal.js#pdf-export)
2. [如何利用 Windows 10 內建 Microsoft Print to PDF 虛擬印表機來將文件列印成 PDF 檔案？](http://key.chtouch.com/ContentView.aspx?P=2431)