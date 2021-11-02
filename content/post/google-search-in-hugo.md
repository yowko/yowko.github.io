---
title: "將 Google自訂搜尋引擎 (Google Custom Search) 搭配 OpenSearch 加至 Hugo 網站中"
date: 2018-10-14T00:10:10+08:00
lastmod: 2021-11-02T00:10:10+08:00
draft: false
tags: ["Google"]
slug: "google-search-in-hugo"
aliases:
    - /2018/10/google-search-in-hugo/
---
## 將 Google自訂搜尋引擎 (Google Custom Search) 搭配 OpenSearch 加至 Hugo 網站中

之前曾在筆記 [如何使用 Blogger APIs Client Library for .NET 匯出 Blogger 文章](/dotnet-blogger-library/) 中提到過去選擇使用 Blogger 當做筆記平台就是看中 Blogger 支援 opensearch ，雖然不是原生啟用，但透過簡單的 plugin 安裝再加上 [怎麼讓網站在輸入網址後按tab 就可以直接搜尋網站內容(OpenSearch)](/opensearch/) 中提到的一些小技巧就可以讓搜尋功能大幅強化，用來彌補個人記憶力差的問題

既然決定轉換平台，當然就要來克服相關問題囉

## Hugo 建議的搜尋方式

我沒有試過所有選項，個人認為除非必要實在不想為了 "站內搜尋" 功能另外使用其他工具 (ex. Grunt、Gulp) 來增加複雜度，而使用 js 來進行內容搜尋在個人測試下準確率實在差強人意，雖然已經使用了 `hugo-lunr-zh` 但在中文及英文的搜尋成功率還是過低，所以放棄

> 詳情請參考 [Search for your Hugo Website](https://gohugo.io/tools/search/)

1. GitHub Gist for Hugo Workflow
2. hugo-elasticsearch
3. hugo-lunr
4. hugo-lunr-zh
5. Github Gist for Fuse.js integration
6. hugo-search-index

## 設定 Google自訂搜尋引擎 (Google Custom Search)

> [Google Custom Search](https://cse.google.com.tw/cse/all)

1. 新增搜尋引擎
    - 填入 `要搜尋的網站`

        > 可以依以下格式來設定

        - 個別網頁：www.example.com/page.html
        - 整個網站：www.mysite.com/*
        - 網站的特定部分：www.example.com/docs/* 或 www.example.com/docs/
        - 整個網域：*.example.com
    - 選擇 `語言`
    - 可修改 `搜尋引擎的名稱`

        > 填完 `要搜尋的網站` 會自動代入

        ![1addgoogle](https://user-images.githubusercontent.com/3851540/46918899-94d46d00-d00a-11e8-9d5d-ad2d30d30a08.png)
2. 調整外觀並取得程式碼
    - 選擇 `編輯搜尋引擎`
    - 下拉選擇想要的搜尋引擎名稱
    - 點選 `外觀和風格`
    - 選擇想要的 `版本配置`
    - 儲存並取得程式碼

    ![2style](https://user-images.githubusercontent.com/3851540/46918900-956d0380-d00a-11e8-99bb-491a1b7ace61.png)

    ![3script](https://user-images.githubusercontent.com/3851540/46918901-956d0380-d00a-11e8-97d8-03998cd2d9b9.png)

## 設定 OpenSearch

> 詳細操作流程請參考過去筆記 [怎麼讓網站在輸入網址後按tab 就可以直接搜尋網站內容(OpenSearch)](/2016/12/opensearch/)，以下內容僅列出步驟

1. 建立 OpenSearch 描述 xml 檔案

    > 格式如下，記得依實際網站內容修改相關資訊

    ```xml
    <OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/">
        <ShortName>Search in Yowko's Notes</ShortName>
        <Description>Search Yowko's Notes via OpenSearch</Description>
        <Tags>Yowko's Notes blog</Tags>
        <Contact>yowko@yowko.com</Contact>
        <Url type="text/html" template="http://blog.yowko.com/search?q={searchTerms}"/>
    </OpenSearchDescription>
    ```

2. 將上述建立的 xml 檔案上傳至網站空間

## 將 OpenSearch 導入 Hugo 網站中

> 引用 OpenSearch 的位置依據 Hugo 的 theme 略有不同，重點就是加在網站的 header 裡，讓每個頁面開啟時都可以順利載入 opensearch 的 xml

```html
<link href='{url}/opensearchdescription.xml' rel='search' title='Content search' type='application/opensearchdescription+xml'/>
```

## Hugo 網站加上 Search 頁籤

> 設定 Search 頁籤與顯示方式依據 Hugo 的 theme 略有不同，請依實際情況自行調整

1. 開啟 Hugo 根目錄的 config.toml
2. 加上 Search 用資料

    ```config
    [[menu.main]]
    name = "站內搜尋"
    weight = 40
    identifier = "search"
    url = "/search"
    ```

## 加入 Search 頁面與搜尋程式碼

> 加入 Search 頁面與搜尋程式碼依據 Hugo 的 theme 略有不同，請依實際情況自行調整

1. 在 content 中建立 `search` 資料夾

    > 與 `posts` 同層

2. 在 `search` 中建立 index.md
    - 設定 metadata

        ```config
        ---
        title: "Search" # 標題
        comment: false # 關閉 disqus，依不同 theme，key 有所不同
        date: 2018-10-14T00:42:34+08:00 
        type: "search"
        ---
        ```

    - 加入搜尋引擎程式碼

        - Hugo 需要透過 Shortcodes 才能將 javascript 加入 .md 中，以下透過 html tag 偷吃步一下
        - 想要由 opensearch 帶參數給 google 站內搜尋，需要在 `gcse:search` 上加入 `queryParameterName="q"`

        ```html
        <div>
        <script type="text/javascript">
        (function() {
            var cx = '{id}';
            var gcse = document.createElement('script');
            gcse.type = 'text/javascript';
            gcse.async = true;
            gcse.src = 'https://cse.google.com/cse.js?cx=' + cx;
            var s = document.getElementsByTagName('script')[0];
            s.parentNode.insertBefore(gcse, s);
        })();
        </script>
        <gcse:search queryParameterName="q"></gcse:search>
        </div>
        ```

## 實際效果

1. 網址列直接搜尋想要的關鍵字

    ![4opensearch](https://user-images.githubusercontent.com/3851540/46918902-956d0380-d00a-11e8-937e-2df366540b7b.png)
2. Hugo 直接整合 goole search 顯示結果

    ![5searchresult](https://user-images.githubusercontent.com/3851540/46918903-956d0380-d00a-11e8-857b-f8c4f3ae31f3.png)

## 心得

一開始為了 Blogger 的 OpenSearch 功能掙扎超久的，主要是筆記對我而言就是用來快速查資料用的，記住關鍵字已經相對不容易了更何況是記住網址或是標題，所以遲遲不敢轉換平台

最後想到可以透過 google search 指定 `site:` 的方式來應急一下，但方便性終究比不上 OpenSearch 幸虧今天花了點時間試出自己使用上不會覺得很卡的方法  紀錄一下以免下次要換平台又卡很久XD

## 參考資訊

1. [如何使用 Blogger APIs Client Library for .NET 匯出 Blogger 文章](/dotnet-blogger-library/)
2. [怎麼讓網站在輸入網址後按tab 就可以直接搜尋網站內容(OpenSearch)](/opensearch/)
3. [Search for your Hugo Website](https://gohugo.io/tools/search/)
4. [Hexo - 自訂站內搜尋(Google Custom Search) | John Wu's Blog](https://blog.johnwu.cc/article/hexo-google-custom-search.html)
