---
title: "如何在 POSTMAN 中針對不同環境設定參數進行測試"
date: 2017-03-06T01:42:34+08:00
lastmod: 2021-10-29T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "postman-parameter-test"
aliases:
    - /2017/03/postman-parameter-test.html
---
## 如何在 POSTMAN 中針對不同環境設定參數進行測試

postman 是我們在開發 api 時的常用工具，不僅是開發者自行測試好用，還有 collection 可以儲存數個測試情境，分享給團隊其他人使用，大幅提高 api 整合的效率。

正因為 postman 如此方便，如果 postman 可以支援在不同環境下使用不同的參數來進行測試，那就更完美了，接著就在 postman 官方部落格上發現了相關的做法，紀錄一下

## 情境說明

1. 測試 url 每個環境是不同的
2. 測試的參數在不同環境也可能不同

## 在 POSTMAN 加入參數

1. Manage Environments
    - 先點一下 齒輪 圖示

        ![1gear](https://cloud.githubusercontent.com/assets/3851540/23515006/f1c834f8-ffa3-11e6-9f5e-cbb5173b6412.png)

    - 選擇 Manage Environments

        ![2MANAGE](https://cloud.githubusercontent.com/assets/3851540/23515008/f1e7225a-ffa3-11e6-9765-aef0fcf0d413.png)

2. 新增環境

    > 新增環境名稱，用來將參數分群
    - ADD

        ![3ADD](https://cloud.githubusercontent.com/assets/3851540/23515009/f1f5195a-ffa3-11e6-99b2-1b6cb501ed1d.png)

    - 自訂環境名稱

        ![4env](https://cloud.githubusercontent.com/assets/3851540/23515010/f1f56efa-ffa3-11e6-9904-c71dd14ffc1d.png)

    - 編輯參數
        - key - value

            ![5param](https://cloud.githubusercontent.com/assets/3851540/23515011/f1f89ecc-ffa3-11e6-8446-b8fa15b21051.png)

3. 新增其他環境及參數
    - 直接複製整組環境及參數再修改

        ![6duplicate](https://cloud.githubusercontent.com/assets/3851540/23515013/f20f7d72-ffa3-11e6-88d5-7e314223f6ed.png)

    - 修改複製的環境及參數
        - 直接點選複製出來的環境

            ![7rename](https://cloud.githubusercontent.com/assets/3851540/23515001/f1816906-ffa3-11e6-9f7d-49b2c050cdcb.png)

        - 修改為需要的名稱及參數值

            ![8changed](https://cloud.githubusercontent.com/assets/3851540/23515003/f1bbe720-ffa3-11e6-8037-cc1cf908b4ea.png)

## 實際使用

1. 以參數撰寫測試
    - 將需要使用參數的地方以 `{{parameter key}}` 取代
    - GET - Url
        - e.g. {{Url}}/api/values

            ![9get](https://cloud.githubusercontent.com/assets/3851540/23515005/f1c80e9c-ffa3-11e6-99db-18354e8df429.png)
    - POST - body

        ![10post](https://cloud.githubusercontent.com/assets/3851540/23515004/f1c6bb8c-ffa3-11e6-8e45-587085611277.png)  

2. 選擇使用的環境
    - 環境 1

        ![11env1](https://cloud.githubusercontent.com/assets/3851540/23515007/f1c888ea-ffa3-11e6-8fba-7b12a49c77c1.png)

    - 環境 2

        ![12env2](https://cloud.githubusercontent.com/assets/3851540/23515012/f202967a-ffa3-11e6-85b2-c7b3bf857ccb.png)

## 可以使用參數的地方

1. URL
2. URL 參數
3. header
4. body - form-data/x-www-form-urlencoded/raw
5. Helper fields

## 參考資料

1. [Setting up an environment with variables](https://www.getpostman.com/docs/environments)
2. [Using environments to switch contexts](https://www.getpostman.com/docs/test_multi_environments)
