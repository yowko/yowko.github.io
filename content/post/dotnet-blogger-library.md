---
title: "如何使用 Blogger APIs Client Library for .NET 匯出 Blogger 文章"
date: 2018-09-15T18:44:34+08:00
lastmod: 2021-11-03T18:44:34+08:00
draft: false
tags: ["套件","csharp"]
slug: "dotnet-blogger-library"
---
## 如何使用 Blogger APIs Client Library for .NET 匯出 Blogger 文章

雖然剛開始決定為自己的開發經驗做些筆記時就已經知道 Blogger 的一些缺點：無法為自訂 domain 加上 https (嘗試過使用 CloudFare ，直到幾個月前已可原生支援)，沒有辦法使用 markdown 語法，JS 與 CSS 相對雜亂，但我個人因為太依賴 OpenSearch 這個功能最後還是選擇使用 Blogger 這個平台

不過隨著 DevOps 觀念愈來愈強化，愈來愈覺得自己用 markdown 寫筆記還要手動轉換為 html 來上架至 Blogger 這件事實在不合理，另外有時為了要調整筆記中的幾個錯字或是排版還要考慮是不是要直接透過 Blogger 的 web 介面修改或是應該改原始 markdown 再重新產 html ，為了 Blogger 排版需要的客製 html (e.g. 拿掉 'h1' 標題 - 避免重複，Blogger 原生就會自動加上 'h1' 標題; 加上 '繼續閱讀' 的連結，避免首頁版面太過冗雜)，都無形中造成負擔

最後剛好看到 蔡煥麟 老師在 FB 的 [貼文](https://www.facebook.com/huanlin.notes/posts/2153043261435153) 中使用的版型很清爽 讓我下定決心來做改版了

## 取得 API key

>需要已有 `Google API Console project`，如果沒有建立過可以至 [Google API Console](https://console.developers.google.com/) 建立

![1consoleproject](https://user-images.githubusercontent.com/3851540/45597333-24cbca80-b9fd-11e8-96e7-45fe59b74d38.png)

* 開啟 [憑證頁面(Credentials page)](https://console.developers.google.com/apis/credentials)

* 建立 API key
  * project -->  `憑證` 選單 --> `憑證` tab --> 建立憑證

        ![2credential](https://user-images.githubusercontent.com/3851540/45597335-24cbca80-b9fd-11e8-938e-4cb141951c24.png)
  * 選擇 API 金鑰

        ![3apikey](https://user-images.githubusercontent.com/3851540/45597336-25646100-b9fd-11e8-8b71-50e7b54930a5.png)
  * 已建立 API 金鑰

        ![4apikeycreated](https://user-images.githubusercontent.com/3851540/45597329-24333400-b9fd-11e8-9101-b1742751a05c.png)

* 取得 API key

    ![5getapikey](https://user-images.githubusercontent.com/3851540/45597330-24333400-b9fd-11e8-986f-fb8a07e92027.png)

## 安裝套件

* Google.Apis.Blogger.v3

    > 我用 `1.35.2.111` 版

  * Package Manager

        ```
        Install-Package Google.Apis.Blogger.v3
        ```
  * .NET CLI

        ```
        dotnet add package Google.Apis.Blogger.v3
        ```

## 取得 Blogger 文章內容

* 程式碼

    ```cs
    //請填 blog id，最簡單的取得進 Blogger 後台網址列就有，也可以由 user 或是 url 查
    var blogId = "blogId";
    
    //建立連線資訊，可以用 OAuth，如果只是需要文章內容， apikey 是最容易的
    var initializer = new Google.Apis.Services.BaseClientService.Initializer()
    {
        ApiKey = "ApiKey"
    };
    
    //使用上述的連線資訊建立 Blogger service 
    Google.Apis.Blogger.v3.BloggerService service = new Google.Apis.Blogger.v3.BloggerService(initializer);
    //取得文章內容
    service.Posts.List(blogId).ExecuteAsync().Items;
    ```

* 預設取得 `10` 筆文章

    ![6tenposts](https://user-images.githubusercontent.com/3851540/45597331-24333400-b9fd-11e8-8a35-17af63bec874.png)

## 自訂取得文章數目

>預設 `10` 筆紀錄，實在太少，就算有提供取得下一頁資料的 token 在沒有程式碼範例下也很難使用

* 程式碼

    ```cs
    //請填 blog id，最簡單的取得進 Blogger 後台網址列就有，也可以由 user 或是 url 查
    var blogId = "blogId";
    
    //建立連線資訊，可以用 OAuth，如果只是需要文章內容， apikey 是最容易的
    var initializer = new Google.Apis.Services.BaseClientService.Initializer()
    {
        ApiKey = "ApiKey"
    };
    
    //使用上述的連線資訊建立 Blogger service 
    Google.Apis.Blogger.v3.BloggerService service = new Google.Apis.Blogger.v3.BloggerService(initializer);
    //透過上面建立的 service 建立文章的 list 物件
    Google.Apis.Blogger.v3.PostsResource.ListRequest request = new Google.Apis.Blogger.v3.PostsResource.ListRequest(service, blogId);
    //指定輸出的文章筆數(service 也有 MaxResults 屬性，但是 readonly)
    request.MaxResults = 500;
    //取得指定筆數文章
    var listRequestResult = request.ExecuteAsync().GetAwaiter().GetResult().Items;
    ```

* 數量愈大，取得時間也會隨之變長

    ![7get500](https://user-images.githubusercontent.com/3851540/45597332-24cbca80-b9fd-11e8-9f2b-74cbb1a8026a.png)

## 將 Blogger 文章匯出為 markdown

原本只是想將 Blogger 的一些 metadata 整批輸出成 hugo style，於是便順便嘗試看看 html 轉 markdown 的效果

* 安裝套件 Html2Markdown

    > 我用 `3.2.1.341` 版

  * Package Manager

        ```
        Install-Package Html2Markdown
        ```
  * .NET CLI

        ```
        dotnet add package Html2Markdown
        ```

* 程式碼

    ```cs
    //請填 blog id，最簡單的取得進 Blogger 後台網址列就有，也可以由 user 或是 url 查
    var blogId = "blogId";
    
    //建立連線資訊，可以用 OAuth，如果只是需要文章內容， apikey 是最容易的
    var initializer = new Google.Apis.Services.BaseClientService.Initializer()
    {
        ApiKey = "ApiKey"
    };
    
    //使用上述的連線資訊建立 Blogger service 
    Google.Apis.Blogger.v3.BloggerService service = new Google.Apis.Blogger.v3.BloggerService(initializer);
    //透過上面建立的 service 建立文章的 list 物件
    Google.Apis.Blogger.v3.PostsResource.ListRequest request = new Google.Apis.Blogger.v3.PostsResource.ListRequest(service, blogId);
    //指定輸出的文章筆數(service 也有 MaxResults 屬性，但是 readonly)
    request.MaxResults = 500;
    //取得指定筆數文章
    var listRequestResult = request.ExecuteAsync().GetAwaiter().GetResult().Items;

    foreach (var item in listRequestResult)
    {
        //透過套件將 html 轉為 markdown
        (new Html2Markdown.Converter()).Convert(item.Content).Dump();
    }
    ```

## 心得

原本想透過這個機會順便調整之前筆記的結構，讓整體閱讀更為舒服，但發現工作結束後的晚上時間只夠改版 15 篇筆記實在太花時間了，後來改變策略：打算將 hugo style 的 metadata 產完，再將之前筆記備份的 markdown 貼進去，直到試了 C# 的 `Html2Markdown` 讓流程更為順暢些，再次感受到略懂程式的喜悅呀

## 參考資訊

1. [Blogger APIs Client Library for .NET](https://developers.google.com/blogger/docs/3.0/api-lib/dotnet)
2. [baynezy/Html2Markdown](https://github.com/baynezy/Html2Markdown)
