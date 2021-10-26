---
title: "網站畫面吃到 SCSS 樣式 ？！"
date: 2017-12-09T23:35:00+08:00
lastmod: 2021-10-26T23:35:49+08:00
draft: false
tags: ["Frontend"]
slug: "scss"
aliases:
    - /2017/12/scss.html
---
## 網站畫面吃到 SCSS 樣式 ？

最近正在如火如塗地開發新專案，眼看著上線時間日漸逼近，網站的功能也慢慢有雛型了，偶爾間幫同事 debug 的過程中發現同事負責開發的頁面看起來怪怪的，為什麼跟樣式跟其他頁面不一樣？！ 不查還好一查竟然找不出真正原因，幸虧過了幾天偶爾間突然有靈感讓我想到原因，以下分享過程跟解決方式

## 異常畫面說明

1. 正常畫面

    ![1normal](https://user-images.githubusercontent.com/3851540/33536644-41123244-d8f1-11e7-9a2c-2ae0276e9737.png)

2. 異常畫面

    ![2stranger](https://user-images.githubusercontent.com/3851540/33536645-413d7f94-d8f1-11e7-967e-0a4a019a45fd.png)

* 異常部份

  * badge 圓角效果過小
  * 文件顏色過深

* 檢查結果

    > 吃到異常 css

    ![3scss](https://user-images.githubusercontent.com/3851540/33536646-416436de-d8f1-11e7-9b9d-3d8093848eb1.png)

## 疑惑部份

1. `.scss` 應該無法直接引用

2. 全站未載入(僅 map 檔用到)

    ![4noscssuse](https://user-images.githubusercontent.com/3851540/33536647-418aed6a-d8f1-11e7-8f3e-36dcfe227097.png)

3. 無法直接取得 `.scss`

    ![5openinother](https://user-images.githubusercontent.com/3851540/33536648-41b15680-d8f1-11e7-890b-7eef6bbdc32c.png)

    ![6404](https://user-images.githubusercontent.com/3851540/33536650-4203101a-d8f1-11e7-87fe-ca964a6ece73.png)

## 問題原因

> 看到上述的第二點是不是已經猜到問題原因了？！沒錯，正是瀏覽器自動解析 .map 檔功能造成的

* 關閉 source map 功能即恢復正常

    1. Chrome --> Settings

        ![7setting](https://user-images.githubusercontent.com/3851540/33796924-1c40056c-dd39-11e7-8e8b-a049b1f0fa3d.png)

    2. Preferences --> Sources --> Enable CSS source maps

        ![8enable](https://user-images.githubusercontent.com/3851540/33796922-1bf129ce-dd39-11e7-8f59-bb06df2f9e13.png)

    3. 使用一般 css

        ![9normalcss](https://user-images.githubusercontent.com/3851540/33796923-1c18d834-dd39-11e7-99ae-da27c4232947.png)

## 心得

這次發現的狀況有點曲折，一開始是畫面異常讓我注意到為什麼是直接被 scss 所影響，雖然很快就發現是重複載入 bootstrap css 造成畫面不正常，但卻搞不清楚為什麼在 View 中引用 bootstrap 就會造成 scss 的載入，也不知道 scss 相關檔案從何處載入，我嘗試修改本機的所有 scss 檔案名稱測試是否會因此引發載入錯誤，並沒有成功

最後不知道怎麼來的靈感才想起可能是 source map 的關係，立馬測試後證實了猜測正確，不過前前後後也花了好幾天才想到，讓我懷念起有專業前端工程師配合的日子呀

## 參考資訊

1. [Sass教學 (24) - Sass 3.3 Source Maps](https://ithelp.ithome.com.tw/articles/10159158)
