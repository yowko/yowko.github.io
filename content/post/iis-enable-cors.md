---
title: "IIS 設定啟用 CORS (Cross-Origin Resource Sharing) - 跨來源資源共用"
date: 2018-02-02T02:06:00+08:00
lastmod: 2021-11-02T02:06:19+08:00
draft: false
tags: ["IIS"]
slug: "iis-enable-cors"
aliases:
    - /2018/02/iis-enable-cors.html
---
## IIS 設定啟用 CORS (Cross-Origin Resource Sharing) - 跨來源資源共用

網站工程師或多或少都曾聽過 CORS (Cross-Origin Resource Sharing) - 跨來源資源共用，甚至遇到相關問題，小弟也不例外，只是以往主要都是調整 ASP.NET MVC 或是 ASP.NET WebAPI 的設定，這次同事遇到的問題則是靜態網頁，而需要調整 IIS 設定，就來看看如何設定 IIS 啟用 CORS 吧

## 關於 CORS (Cross-Origin Resource Sharing) - 跨來源資源共用

以下內容節錄自 [跨來源資源共享（CORS）](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS)，完整內容請參考 [跨來源資源共享（CORS）](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS)

```txt
跨來源資源共享（Cross-Origin Resource Sharing (CORS)）是一種使用額外 HTTP 標頭令目前瀏覽網站的用戶代理取得訪問其他來源（網域）伺服器特定資源權限的機制。當用戶代理請一個不是目前文件來源——例如來自於不同網域（domain）、通訊協定（protocol）或通訊埠（port）的資源時，會建立一個跨來源 HTTP 請求（cross-origin HTTP request）。
舉個跨來源請求的例子：`http://domain-a.com` HTML 頁面裡面一個 <img> 標籤的 src 屬性載入來自 http://domain-b.com/image.jpg 的圖片。現今網路上許多頁面所載入的資源， CSS 樣式表、圖片影像、以及指令碼（script）都來自與所在位置分離的網域，如內容傳遞網路（content delivery networks, CDN）。
基於安全性考量，程式碼所發出的跨來源 HTTP 請求會受到限制。例如，XMLHttpRequest 及 Fetch 都遵守同源政策（same-origin policy）。這代表網路應用程式所使用的 API 除非使 CORS 標頭，否則只能請求與應用程式相同網域的 HTTP 資源。
```

<img src="https://mdn.mozillademos.org/files/14295/CORS_principle.png" alt="cors" style="background-color: white;">

```txt
跨來源資源共享（Cross-Origin Resource Sharing，簡稱 CORS）機制提供了網頁伺服器跨網域的存取控制，增加跨網域資料傳輸的安全性。現代瀏覽器支援在 API 容器（如 XMLHttpRequest 或 Fetch）中使用 CORS 以降低跨來源 HTTP 請求的風險。
```

## 實際情境說明

* 透過 ajax 取得其他 domain 的圖片

    ```JS
    $(function () {
        $.ajax({
            method: "GET",
            url: "http://localhost:8011/test.png",
        });
    })
    ```

* IIS 未設定啟用 CORS 出現錯誤
    * 錯誤訊息

        ```log
        Failed to load http://localhost:8011/test.png: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:2165' is therefore not allowed access.
        ```

    * 錯誤截圖

        ![1nocors](https://user-images.githubusercontent.com/3851540/35694522-99a0fa72-07bc-11e8-8309-a2b257a0edef.png)

## IIS 啟用 CORS

1. 選擇目標 IIS 站台 --> double click `Http Response Header`

    ![2httpheader](https://user-images.githubusercontent.com/3851540/35694524-99d252de-07bc-11e8-8c6b-8c547d575534.png)

2. Add

    ![3addheader](https://user-images.githubusercontent.com/3851540/35694525-9a017e1a-07bc-11e8-9314-aa5a82ca5a02.png)

    * Name：`Access-Control-Allow-Origin`
    * Value：`*` 表示對所有網站開放，或可指定特定來源網站(僅能設定一組)

        ![4specificsite](https://user-images.githubusercontent.com/3851540/35694527-9a2ee35a-07bc-11e8-8cda-e34acc8d5a24.png)

        * 僅能設定一組

            ![5multipleerror](https://user-images.githubusercontent.com/3851540/35694528-9a5b05d4-07bc-11e8-8f48-df3039f252e0.png)

## 心得

小小設定影響甚鉅呀，ASP.NET MVC 或是 ASP.NET WebAPI 網站也可以透過 IIS 設定來啟用 CORS，不過 IIS 設定不支援正向表列多組 domain，但如果是單一 domain 或是 `*` 對於所有來源開放的情境，使用 IIS 來設定就非常便利了

同事的問題其實不是 IIS 設定，只是自己在模擬過程中發現不熟悉相關設定，隨手紀錄一下，至於同事的問題就待下篇繼續探討

## 參考資訊

1. [跨來源資源共享（CORS）](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS)
