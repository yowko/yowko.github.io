---
title: "為 Hugo 加上 InstantClick"
date: 2018-11-01T00:42:34+08:00
lastmod: 2018-11-01T00:42:34+08:00
draft: false
tags: ["Hugo","JavaScript","套件"]
slug: "hugo-js"
---
# 為 Hugo 加上 InstantClick
前幾天 Larry 哥閒聊問到 Hugo 在引用其他 js 套件上的支援度時舉了 InstantClick.js 的例子，無奈小弟書讀得不夠多，沒用過 InstantClick.js，Larry 哥立馬提供了自己的紀錄 ([InstantClick - JS library to make your website instant](http://larrynung.github.io/2017/08/02/instantclick-js-library-to-make-your-website-instant/) 及 [Hexo - Speed up with InstantClick](http://larrynung.github.io/2017/08/03/Hexo-Speed-up-with-InstantClick/))讓我學習

當下雖然沒有十足地把握達成，但憑藉 Hugo 高人氣跟之前整合 Google Custom Search 的經驗，相信應該是沒有問題，不過口說無憑就來看看如何達成目的吧

## 調整步驟

> **調整內容會因為使用的 theme 不同而有所差異，概念大致方向是相同的**，以下使用 [yowko/hugo-theme-even-more](https://github.com/yowko/hugo-theme-even-more) 當做範例

1. 找到 theme 使用 js 的範本位置

    > 這個步驟見人見智，我個人觀點會站在盡量配合原始 theme 的架構修改

     以我使用的 theme 為例，範本位置為 `layouts/partials/scripts.html`

2. 加入引用的 js

    > 以我所使用的 theme 為例，原生就提供了 CDN 與 self-host 的方式，所以需要個別針對不同模式做調整

    - CDN
        - 在網站 root 下的 `config.toml` 中加入 CDN 位置

            ```toml
            instantclick = '<script src="https://cdnjs.cloudflare.com/ajax/libs/instantclick/3.0.1/instantclick.min.js" crossorigin="anonymous" data-no-instant></script>'
            ``` 
        - 在步驟 1 的 js 引用範本中加入引用 js
            
            ```
            {{ .Site.Params.publicCDN.instantclick | safeHTML }}
            ```
    - self-host
        - 將 js 檔加入 theme 的資料中

            > 以我的例子是 `lib/instantclick/instantclick-3.0.1.min.js` 
        - 在步驟 1 的 js 引用範本中加入引用 js

            ```
            <script type="text/javascript" src="{{ "lib/instantclick/instantclick-3.0.1.min.js" | relURL }}" data-no-instant></script>
            ```
    
    * 實際修改的內容前後比較
        
        ![1addjs](https://user-images.githubusercontent.com/3851540/47866870-e9654e00-de3a-11e8-82c0-11ec0e193cc9.png)

3. 設定啟用 instantclick
    
    > 在步驟 1 的 js 引用範本中加上啟用 instantclick 的 script

    ```
    <!--instantclick-->
    <script data-no-instant>InstantClick.init();</script>
    ```
4. 實際效果

    > 滑鼠移至連結上方，instantclick 就會預先去抓取目標網址的內容

    ![instantclick](https://user-images.githubusercontent.com/3851540/47866871-e9654e00-de3a-11e8-9468-3fd4b0348441.gif)

## 心得

之前為了加 Google Custom Search 時發現 Hugo 在 js 的處理有自己的一套語法，但說實話我不太想為了筆記還要特別去記語法，所幸這次修改只動了 theme 中 template 以及網站的 config 不需要使用 Hugo 特定語法

實際加上功能的時間很短，反而是一開始要找到引用 js 的 partial 頁面還比較花時間

# 參考資訊
1. [InstantClick.js](http://instantclick.io/)
2. [InstantClick - JS library to make your website instant](http://larrynung.github.io/2017/08/02/instantclick-js-library-to-make-your-website-instant/)
3. [Hexo - Speed up with InstantClick](http://larrynung.github.io/2017/08/03/Hexo-Speed-up-with-InstantClick/)
