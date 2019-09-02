---
title: "[JMeter] 取得 JSON Token 做為 Http Request Header 參數"
date: 2019-09-02T01:30:00+08:00
lastmod: 2019-09-02T01:30:31+08:00
draft: false
tags: ["JMeter"]
slug: "jmeter-json-parameter"
---

## [JMeter] 取得 JSON Token 做為 Http Request Header 參數

一般公開對外的 REST API 基於安全性考量都會有認驗證機制，做法很多，其中一種做法就是透過使用指定格式來取得 token，後續的資料交換就改以 token 為識別是否為合法 request 並提供有效 response 的基準。因此測試情境也就需要有相應的處理：先取得 token 才能測試目標 api

今天簡單紀錄一下，取得 token 時內容沒有加密的做法，當做後續紀錄遇到加密 response 的比較基準

## 基本環境說明

1. macOS Mojave 10.14.6
2. JMeter 5.1.1
3. 測試用 api ： [reqres.in](https://reqres.in/)

## JMeter 設定

1. Test Plan 右鍵 --> Add --> Threads (Users) --> Thread Group

    ![1addthread](https://user-images.githubusercontent.com/3851540/64123783-e11dcc80-cdd7-11e9-9259-e8d7229b94a1.png)

2. 建立 Login Http Header Manager

    > 用來管理取得 token 的 request header

    ![2header](https://user-images.githubusercontent.com/3851540/64123784-e11dcc80-cdd7-11e9-92fc-7599b9ec29f1.png)

    ![2loginheader](https://user-images.githubusercontent.com/3851540/64123785-e1b66300-cdd7-11e9-8e80-cfede5d482f9.png)

3. 建立 Login HTTP Request

    > 用來實際取得 token，測試 api 用法請參考 [reqres.in](https://reqres.in/) 說明

    ![3request](https://user-images.githubusercontent.com/3851540/64123789-e1b66300-cdd7-11e9-8544-1ff01b486a2c.png)

    ![3httprequest](https://user-images.githubusercontent.com/3851540/64123788-e1b66300-cdd7-11e9-9f00-79a3db2812be.png)

4. 建立 JSON Extractor

    > 取得回傳的 JSON 內容，並設定為 JMeter 變數

    ![4jsonextractor](https://user-images.githubusercontent.com/3851540/64123790-e1b66300-cdd7-11e9-8b76-88af7d309dcb.png)

    ![5extractoresetting](https://user-images.githubusercontent.com/3851540/64123791-e24ef980-cdd7-11e9-8230-6230deeaaddc.png)

    - Names of created variables 為指定 JMeter 的變數名稱
    - JSON Path expression 為目標 JSON 屬性名稱

5. 再次建立 Http Header Manager

    > 用來管理實際執行目標 api 的 header 設定, 並使用上方 JSON Extractor 設定的參數

    ![6header](https://user-images.githubusercontent.com/3851540/64123792-e24ef980-cdd7-11e9-973b-1965d76a78a8.png)

6. 建立測試目標 HTTP Request

    > 實際執行測試目標

    ![7request](https://user-images.githubusercontent.com/3851540/64123793-e24ef980-cdd7-11e9-8c40-fef5d71dbdf7.png)

7. 加入 View Results Tree

    > 用來檢視各步驟的執行結果

    ![8viewresult](https://user-images.githubusercontent.com/3851540/64123794-e2e79000-cdd7-11e9-9c6f-5c7661dcdbe0.png)

8. 實際執行測試

    可以從 View Results Tree 看到 HTTP Request - Login 取得 token : `QpwL5tke4Pnpja7X4`，並可以看到 HTTP Request 的 request header 也正確使用 `Authorization: Bear QpwL5tke4Pnpja7X4`

    ![9gottoken](https://user-images.githubusercontent.com/3851540/64123795-e2e79000-cdd7-11e9-9ae4-97cb0f8698fa.png)

    ![10correctheader](https://user-images.githubusercontent.com/3851540/64123797-e2e79000-cdd7-11e9-8eda-d64a03601562.png)

## 心得

簡單的設定，沒有複雜的操作，上手難度不高，如果真實環境可以這麼簡單想必工作起來一定輕鬆寫意不少，不過也就是因為可以面對這麼多變的需求才顯得程式工作的挑戰性跟有趣

## 參考資訊

1. [reqres.in](https://reqres.in/)
2. [JMeter解析JSON內容，動態取得變數](https://tpu.thinkpower.com.tw/tpu/articleDetails/620)
