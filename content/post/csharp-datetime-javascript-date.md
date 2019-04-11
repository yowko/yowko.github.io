---
title: "C# DateTime 轉 JavaScript Date (依使用者偏好區域來顯示時間)"
date: 2018-02-05T00:17:00+08:00
lastmod: 2018-10-03T00:17:04+08:00
draft: false
tags: ["套件","C#","JavaScript"]
slug: "csharp-datetime-javascript-date"
aliases:
    - /2018/02/c-datetime-javascript-date.html
---
# C# DateTime 轉 JavaScript Date (依使用者偏好區域來顯示時間)
同事負責的頁面會有不同時區的 user 來瀏覽，時間類型的顯示會直接影響 user 的使用者體驗，尤其是各式公告跟有時效性的操作更是需要特別留意

主要的需求就是用 C# 從 DB 取出 DateTime 資料，接著顯示在網頁上，而 DateTime 資料需要依據使用者所在時區來顯示

以下紀錄一下個人想到的做法，再跟同事交流看看可以怎麼改善

## 概念

1.  C# 取得 DateTime 資料傳給 JavaScript
2.  JavaScript 將原 DateTime 轉為 user 所在時區時間
3.  將轉換完成的時間呈現在網頁上


## 1. C# 取得 DateTime 資料傳給 JavaScript

> 這是整個需求中最核心的部份，最常遇到問題的就是這邊

1.  將目標時間與 js 的啟始時間 (1970/1/1 0:0:0) 兩者時間差轉為毫秒，傳給 js ，再透過 `new Date(總毫秒數)` 取得
    *   c#

        ```cs
        var _target=sourceDatetime.Subtract(new DateTime(1970,1,1,0,0,0,DateTimeKind.Utc)).TotalMilliseconds;
       ```

    *   js

        ```js
        var dt = new Date(_target);
        ```

2.  將 C# DateTime 轉為 js 可直接的格式

    *   C# (下列兩者皆可)

        *   ToString("R") : `Fri, 02 Feb 2018 15:30:14 GMT`

            ```cs
            sourceDatetime.ToString("R");
            ```

        *   ToString("u"):`2018-02-02 15:30:14Z`

            ```cs
            sourceDatetime.ToString("u");
            ```

    *   js

        ``js
        var dt = new Date(targetDatetime);
        ```

3.  搭配 moment.js
    *   C# (與 js 相同)

        *   ToString("R") : `Fri, 02 Feb 2018 15:30:14 GMT`

            ```cs
            sourceDatetime.ToString("R");
            ```

        *   ToString("u"):`2018-02-02 15:30:14Z`

            ```cs
            sourceDatetime.ToString("u");
            ```

    *   moment.js

        ```js
        var dt = moment(targetDatetime);
        ```

## 2. JavaScript 將原 DateTime 轉為 user 所在時區時間

1.  js

    ```js
    var result=dt.toLocaleString();
    ```

2.  moment.js

    ```js
    var result=dt.local().format();
    ```

## 3. 將轉換完成的時間呈現在網頁上

```
 $('selector').html(result);
 ```

## 其他

就上面看來，其實使用 moment.js 效益並不明顯，但如果想要指定輸出的 format 這時候就缺不了 moment.js 的便利性了，說實話如果只用 js 要格式化輸出可是件苦差事呀

*   moment.js 格式化

    ```js
    var result=dt.local().format('DD/MM/YYYY HH:mm')
    ```

## 完整程式碼

1.  html(cshtml)

    > 由 Server render 指定的 datetime string

    *   `R`

        ```html
        <div id="waitchange" data-val="Fri, 02 Feb 2018 15:30:14 GMT"></div>
        ```

    *   `u`

        ```html
        <div id="waitchange" data-val="2018-02-02 15:30:14Z"></div>
        ```

*   JavaScript

    > 截圖內容說明：
    > 
    >     *   `TestDateTime` 表示 DB 儲存 UTC 時間
    >     *   `TestDateTime (Server Time)` 表 UTC 時間`.ToLocalTime()`
    >     *   `TestDateTime (Client Time)` 為 JS 處理結果

    *   原生 js

        ```js
        var _element = $('#waitchange');
        var dt = new Date($(_element).data('val'));
        var result=dt.toLocaleString();
        $(_element).html(result);
        ```

        ![1purejs](https://user-images.githubusercontent.com/3851540/35779555-bd6e3632-0a09-11e8-8e36-4c9aea51e649.png)

    *   使用 moment.js

        ```js
        var _element = $('#waitchange');
        var dt = moment($(_element).data('val'));
        var result=dt.local().format();
        $(_element).html(result);
        ```

        ![2momentjs](https://user-images.githubusercontent.com/3851540/35779492-d6e250b8-0a08-11e8-9157-947e35fdc1f8.png)

    *   使用 moment.js 格式化輸出

        ```js
        var _element = $('#waitchange');
        var dt = moment($(_element).data('val'));
        var result=dt.local().format('DD/MM/YYYY HH:mm');
        $(_element).html(result);
        ```

        ![3momentjsformat](https://user-images.githubusercontent.com/3851540/35779493-d708ddd2-0a08-11e8-8bac-38d7c46bc150.png)

## 心得

看似簡單的需求，其實背後還隱藏著一些小技巧：

1.  C# 與 js 之間的 datetime 轉換，有特定格式的要求
2.  js date 顯示的格式微調，有 moment.js 會簡單得多


這讓我想起以前有位大神(我忘了是誰XD)說過的話：`你不知道語言特性的限制或是沒有用 library 並不代表你做不出來，但別人就是贏你在這些知識所節省下來的時間` 雖然要記得所有語言限制跟認識所有 library 太不切實際，但至少希望可以在每次遇到同樣問題時不用花費一樣的時間在重新來過上

# 參考資訊

1.  [ateTime to javascript date](https://stackoverflow.com/questions/2404247/datetime-to-javascript-date)
2.  [DateTime.ToString 方法 (String)](https://msdn.microsoft.com/zh-tw/library/zdtaw1bw%28v=vs.110%29.aspx)
3.  [JavaScript Date Formats](https://www.w3schools.com/js/js_date_formats.asp)
4.  [Invalid Date in Firefox using .toLocaleString() as well as moment.js](https://stackoverflow.com/questions/44929926/invalid-date-in-firefox-using-tolocalestring-as-well-as-moment-js)
5.  [Moment.js Documentation](https://momentjs.com/docs/)
