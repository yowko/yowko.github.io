---
title: "ASP.NET MVC 送出含有 `<` 的表單內容時出現錯誤"
date: 2017-06-17T00:42:00+08:00
lastmod: 2018-09-22T00:42:09+08:00
draft: false
tags: ["ASP.NET MVC","ASP.NET Web API"]
slug: "aspnet-mvc-post-html"
aliases:
    - /2017/06/aspnet-mvc-post-html.html
---
# ASP.NET MVC 送出含有 `<` 的表單內容時出現錯誤
這是同事今天遇到的問題，當看到錯誤訊息時，我心中百感交集，想當年沒日沒夜趕專案時，這個錯誤訊息可是三天兩頭就會看到一次：基本的 CMS 後台常常搭配的就是 HTML 編輯器，MVC 一送出就會出現一樣熟悉的錯誤訊息。

雖然是錯誤訊息，但像是久未謀面的老朋友，再次相見也時隔二三年了，心中竟有些懷念，也許懷念的不是一樣的錯誤訊息而是懷念當年錯誤訊息代表的時空背景；除了懷念，也充滿了感嘆，當年看到這個錯誤訊息，腦袋都還沒完全反應過來，手已經敲著鍵盤把問題解決了XD 時隔三年的今日，我竟連關鍵字都想不起來，可真是歲月不饒人呀，為了避免下次又想不起來 只得認份地筆記一下

## 錯誤訊息

1. 錯誤訊息

    ``` 
    Server Error in '/' Application.

    A potentially dangerous Request.Form value was detected from the client (value="<dsfsd132").

    Description: ASP.NET has detected data in the request that is potentially dangerous because it might include HTML markup or script. The data might represent an attempt to compromise the security of your application, such as a cross-site scripting attack. If this type of input is appropriate in your application, you can include code in a web page to explicitly allow it. For more information, see http://go.microsoft.com/fwlink/?LinkID=212874. 

    Exception Details: System.Web.HttpRequestValidationException: A potentially dangerous Request.Form value was detected from the client (value="<dsfsd132").

    Source Error:
    ```
2. 錯誤截圖
    
    ![1errormsg](https://user-images.githubusercontent.com/3851540/27235954-88eab5b4-52f5-11e7-977b-6eabaf562d70.png)

> 大意就是偵測到 user 輸入的資料中出現了 html 語法或是 javascript，為了避免 XSS 攻擊，所以拒絕網頁執行

## 原始程式碼

1.  MVC 的 controller

    ```cs
    public ActionResult Values(string value, int id)
    {
        return Content($"{id}-{value}");
    }
    ```

2.  網頁

    ```cs
    @using (Html.BeginForm("values", "Home", FormMethod.Post))
    {
        @Html.TextBox("id",11);
        @Html.TextBox("value");
        <input type="submit" value="Submit" class="btn btn-primary"/>
    }
    ```

## 如何調整

*   指定 Action 不檢查

    > 在 Action 上套用 `[ValidateInput(false)]`

    ```cs
    [ValidateInput(false)]
    public ActionResult Values(string value, int id)
    {
        return Content($"{id}-{value}");
    }
    ```

## 如何優化

直接在 action 上套用 `[ValidateInput(false)]` 是快速又有效的，只是套用 `[ValidateInput(false)]` 後會讓整個 Action 都不做 XSS 的 HTML tag 檢查，這樣是有風險的，正確的做法：<span style="color:red">僅針對需要的欄位屬性開放</span>，不該是無差別式地全部開放

*   修改方式
    *   將所有傳遞參數統整為一個 class

        ```cs
        public ActionResult Values(string value, int id)
        ```

    * 改為下列寫法

        ```cs
        public ActionResult Values(PstData model)
        public class PstData
        {
            public int id { get; set; }
            public string value { get; set; }
        }
        ```

    *   在需要開放的屬性上套件 `[AllowHtml]`

        ```cs
        public class PstData
        {
            public int id { get; set; }
            [AllowHtml]
            public string value { get; set; }
        }
        ```

## 心得

順手測試了 ASP.NET Web Api，並沒有相同檢查，個人推測 Web API 沒有頁面，不會發生 XSS 的問題，就是資料存取的角色，所以沒有特別檢查

雖然是個曾經熟悉現在又顯得陌生的問題，但重新複習起來也別有一番風味呀

# 參考資訊

1.  [Request Validation in ASP.NET](https://msdn.microsoft.com/en-us/library/hh882339.aspx)
