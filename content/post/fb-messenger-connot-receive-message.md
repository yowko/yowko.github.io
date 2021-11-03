---
title: "FB Messenger 回應 '此用戶目前無法收到你的訊息'"
date: 2017-01-11T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["facebook"]
slug: "fb-messenger-connot-receive-message"
aliases:
    - /2017/01/fb-messenger-connot-receive-message.html
---
## FB Messenger 回應 "此用戶目前無法收到你的訊息"

在測試 fb messenger 的過程中，為了不要讓其他人看到粉絲團，在一番胡亂設定下，竟然讓粉絲團的 fb messenger 出現 `此用戶目前無法收到你的訊息`，再經過一番測試後終於找到原因。

## 錯誤訊息

> 此用戶目前無法收到你的訊息

![1.noreceived](https://cloud.githubusercontent.com/assets/3851540/21815625/5616dbf0-d798-11e6-8d35-cfbe41cc4061.png)

## 檢查 FB Application 設定

facebook 開發人員網站 [facebook for developers](https://developers.facebook.com/)

1. `Messenger` 是否啟用

    ![enablemessenger](https://cloud.githubusercontent.com/assets/3851540/21815636/61043774-d798-11e6-8e82-4bc924e2d534.png)

2. `Messenger` 設定
    - token

        ![tokengot](https://cloud.githubusercontent.com/assets/3851540/21815695/8b393e7c-d798-11e6-961b-3f4ed08124fb.png)

## 檢查粉絲專頁設定

> 需啟用`顯示訊息按鈕，以允許他人私下與我的粉絲專頁聯絡`

![pagesetting](https://cloud.githubusercontent.com/assets/3851540/21815626/561a1fc2-d798-11e6-80d6-a9ec8d379be8.png)

## 參考資料

1. [Facebook Messenger Bot 需要申請什麼呢？](http://blog.yowko.com/2016/12/facebook-messenger-bot.html)
