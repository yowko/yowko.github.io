---
title: "使用 Moment.js 來簡化 JavaScript 的時間處理"
date: 2018-01-20T22:22:00+08:00
lastmod: 2018-10-03T22:22:30+08:00
draft: false
tags: ["套件","Frontend","JavaScript"]
slug: "moment-js"
aliases:
    - /2018/01/moment-js.html
---
# 使用 Moment.js 來簡化 JavaScript 的時間處理
專案的前端頁面在 user 要求加速整個流程下，使用了一套 library - [Date Range Picker](http://www.daterangepicker.com/) 方便使用者可以在同一個 input 中同時選擇 start date 與 end date 也讓整體操作獲得簡化，整體改善獲得 user 的好評

故事開頭似乎跟筆記的標題沒什麼關係？！ 因為使用了 [Date Range Picker](http://www.daterangepicker.com/) 來處理需要同時選擇 start date 與 end date 的需求，而為了讓頁面呈現統一也使用 [Date Range Picker](http://www.daterangepicker.com/) 來處理單一日期的選擇，在設定單一日期選擇器的過程中發現 [Date Range Picker](http://www.daterangepicker.com/) 送出的日期一樣是 start date 與 end date，並且相當貼心的已經處理成 start date 為選擇日期的 `00:00:00` 而 end date 為選擇日期的 `23:59:59` 讓後端程式可以少做一些處理 非常方便，需是乎就打算來學習一下 [Date Range Picker](http://www.daterangepicker.com/) 是如何處理 js 的時間問題也才發現今天主角 - [Moment.js](https://momentjs.com/) 的強大功能

今天就來簡單地紀錄一下自己常用的 Moment.js 功能吧

## 功能介紹

1.  取得當下時間
    *   js

        ```js
        new Date()
        ```

        ![1nowjs](https://user-images.githubusercontent.com/3851540/35184288-9d60e682-fe2e-11e7-9790-b477dd7e7282.png)

    *   c#

        ```cs
        DateTime.Now
        ```

        ![2nowcs](https://user-images.githubusercontent.com/3851540/35184286-9d0d4e28-fe2e-11e7-9017-88243237b47e.png)

    *   moment.js

        ```js
        moment().format()
        ```
        
        ![3nowmoment](https://user-images.githubusercontent.com/3851540/35184287-9d377086-fe2e-11e7-87d5-b86140ee1978.png)

2.  時間格式轉換

    > '2018/01/01 00:00:00'

    *   js

        ```js
        var now = new Date();
        var year = now.getFullYear();
        var month = ("0" + (now.getMonth() + 1)).slice(-2);
        var day = ("0" + (now.getDate() + 1)).slice(-2);
        var hour = ("0" + (now.getHours() )).slice(-2);
        var minute = ("0" + now.getMinutes() ).slice(-2);
        var second = ("0" + now.getSeconds() ).slice(-2);
        year + '/' + month + '/' + day + ' ' + hour + ':' + minute+ ':' + second
        ```

    *   c#

        ```cs
        DateTime.Now.ToString("yyyy/MM/dd HH:mm:ss")
        ```
    *   moment.js

        ```js
        moment().format('YYYY/MM/DD HH:mm:ss')
        ```
3.  時間處理

    > 加一天

    *   js

        ```js
        var now = new Date();
        var year = now.getFullYear();
        var month = ("0" + (now.getMonth())).slice(-2);
        var day = ("0" + (now.getDate() + 1)).slice(-2);
        var hour = ("0" + (now.getHours())).slice(-2);
        var minute = ("0" + now.getMinutes()).slice(-2);
        var second = ("0" + now.getSeconds()).slice(-2);
        new Date(year, month, day, hour, minute, second);
        ```

    *   c#

        ```cs
        DateTime.Now.AddDays(1)
        ```
    *   moment.js

        ```js
        moment().add(7, 'days');
        ```

        ```js
        moment().add(7, 'd');
        ```
        
        > 使用介紹 [Add](https://momentjs.com/docs/#/manipulating/add/)

        |Key|Shorthand|
        |--- |--- |
        |years|y|
        |quarters|Q|
        |months|M|
        |weeks|w|
        |days|d|
        |hours|h|
        |minutes|m|
        |seconds|s|
        |milliseconds|ms|


4.  取得當天開頭時間及結束時間

    *   js

        > 請原諒小弟 js 略過, 個人能力不足寫不出來

    *   c#

        ```cs
        DateTime.Today.ToString("yyyy/MM/dd HH:mm:ss");
        DateTime.Today.AddDays(1).AddMilliseconds(-1).ToString("yyyy/MM/dd HH:mm:ss");
        ```

        ![4srartendcs](https://user-images.githubusercontent.com/3851540/35184399-09ca72a6-fe30-11e7-977a-180875c1945a.png)

    *   moment.js

        ```js
        moment().startOf('day').format('YYYY/MM/DD HH:mm:ss')
        moment().endOf('day').format('YYYY/MM/DD HH:mm:ss')
        ```

        ![5startendmoment](https://user-images.githubusercontent.com/3851540/35184400-09f4f102-fe30-11e7-89c1-51c106927ecf.png)

## 心得

moment.js 功能很多，使用上很方便，最重要的是文件算是滿完整的，讓實際導入時的問題少很多，至於其他功能：指定時間、顯示格式、時間比較、多語言...各式操作就不一一紀錄，請直接參考 [Moment.js](https://momentjs.com/)

而在實際紀錄 [Moment.js](https://momentjs.com/) 的過程中讓我想起幾年前，好像曾經聽同事分享過這個 library 很好用，只是當時剛好沒有實際使用的契機也就沒有太深印象，想不到最後兜了一大圈才真正體會到當時同事說的好用

# 參考資訊

1.  [Date Range Picker](http://www.daterangepicker.com/)
2.  [Moment.js](https://momentjs.com/)
