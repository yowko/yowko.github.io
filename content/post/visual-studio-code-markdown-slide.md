---
title: "如何在 Visual Studio Code 中快速使用 Markdown 建立簡報"
date: 2017-04-24T01:00:00+08:00
lastmod: 2021-10-14T02:01:59+08:00
draft: false
tags: ["套件","Tools"]
slug: "visual-studio-code-markdown-slide"
aliases:
    - /2017/04/visual-studio-code-markdown-slide.html
---
## 如何在 Visual Studio Code 中快速使用 Markdown 建立簡報

今天看到[保哥分享](https://www.facebook.com/groups/augularjs.tw/permalink/1588293701180994/) vscode-reveal 套件可以直接在 visual studio code 中使用 reveal.js 來開啟 markdown 檔做成簡報，感覺很像之前文章 [用 Markdown 與 Reveal.js 來製作簡報](/markdown-revealjs) 分享的做法，所以立馬吸引了我的目光，因此特別來體驗比較一下

## Visual Studio Code 安裝 `vscode-reveal` 套件(擇一即可)

1. 使用指令安裝

    * ctrl+p

        > ext install vscode-reveal

2. 使用介面安裝

    * Extensions --> Search 'vscode-reveal' --> Install
        ![1install](https://cloud.githubusercontent.com/assets/3851540/25314950/bc22c566-287f-11e7-9783-a500b015e823.png)

## Visual Studio Code 如何開啟簡報(擇一其可)

>先開啟 markdown 檔案

1. 使用快捷鍵
    * Ctrl+Shift+P --> Revealjs: Show presentation by side

        ![3showside](https://cloud.githubusercontent.com/assets/3851540/25314951/bc22f7d4-287f-11e7-9b24-d8800570e2fc.png)
2. 使用介面
    * View --> Command Palette... --> Revealjs: Show presentation by side

        ![2command](https://cloud.githubusercontent.com/assets/3851540/25314952/bc247636-287f-11e7-9239-243b76ef0757.png)

        ![3showside](https://cloud.githubusercontent.com/assets/3851540/25314951/bc22f7d4-287f-11e7-9b24-d8800570e2fc.png)

## 設定參數

* 參數內容比照 [reveal.js](https://github.com/hakimel/reveal.js)

* 參數的設定直接寫在 markdown 檔的開頭當做第一頁

    ```md
    ---
    progress: true
    slideNumber: true
    history: true
    ---
    簡報開始
    ---
    ```

    Name|Description|Default
    :---|:---|:---
    notesSeparator|註解標記|note:
    separator|換頁符號|`---`
    verticalSeparator|垂直簡報分割符號|`--`
    theme|佈景主題：<br/>black<br/>    white<br/>    league<br/>    beige<br/> sky<br/>    night<br/>    serif<br/>    simple<br/>    solarized|black
    highlightTheme|高亮主題|Zenburn
    controls|是否顯示換頁控制器|true
    progress|是否顯示簡報進度條|true
    slideNumber|是否顯示簡報頁數|false
    history|簡報換頁是否保留至瀏覽器紀錄|false
    keyboard|是否啟用鍵盤來換頁|true
    overview|是否啟用簡報全覽|true
    center|是否將內容垂直置中|true
    touch|是否啟用觸控功能來換頁true
    loop|循環顯示簡報|false
    rtl|改變簡報順序成由右至左|false
    shuffle|每次換頁都隨機載入|false
    fragments|是否開啟區塊逐步顯示|true
    embedded|是否開啟 embedded mode|false
    help|按下 `?` 是否在最上層顯示相關資訊|true
    showNotes|是否直接顯示註解內容|false
    autoSlide|自動換頁毫秒數，0 表示停用自動換頁<br/>可以被簡報的 data-autoslide 屬性複寫|0
    autoSlideStoppable|當使用者屬入時停用自動換頁|true
    mouseWheel|是否啟用滑鼠滾輪來換頁|false
    hideAddressBar|在手機上隱藏 url|true
    previewLinks|內嵌連結內容|false
    transition|轉場特效：<br/>none<br/>fade<br/>slide<br/>convex<br/>concave<br/>zoom|slide
    transitionSpeed|轉場速度：<br/>    default<br/>    fast<br/>    slow|default
    backgroundTransition|背景的轉場特效：<br/>    none<br/>   fade<br/>    slide<br/>    convex<br/>    concave<br/>    zoom|fade
    viewDistance|預先載入的頁數|3
    parallaxBackgroundImage|視差背景圖|-
    parallaxBackgroundSize|視差背景大小 <br/>    (CSS syntax, e.g. 2100px 900px)|-
    parallaxBackgroundHorizontal|每頁簡報視差背景水平移動的距離|-
    parallaxBackgroundVertical|每頁簡報視差背景垂直移動的距離|-

## 心得

1. 不需另外設架網頁伺服器
2. 不需自己調整載入的 js 、css 、md 檔
3. 參數設定方式較簡潔
4. 在 visual studio code 中顯示簡報比較麻煩，有內容更新都要關閉預覽視窗再重開才會生效(瀏覽器只需 refresh)
5. 一樣可以匯出 pdf ，詳細做法請參考 [如何將 Reveal.js 簡報匯出成 PDF](/revealjs-pdf)

**總言之非常推薦改用這個做法**

## 參考資訊

1. [vscode-reveal(VSC markeplace)](https://marketplace.visualstudio.com/items?itemName=evilz.vscode-reveal)
2. [vscode-reveal(GitHub)](https://github.com/evilz/vscode-reveal)
3. [reveal.js](https://github.com/hakimel/reveal.js)
4. [用 Markdown 與 Reveal.js 來製作簡報](/markdown-revealjs)
5. [如何將 Reveal.js 簡報匯出成 PDF](/revealjs-pdf)
