---
title: "Visual Studio JavaScript 編輯視窗出現 ECMAScript 錯誤提示"
date: 2017-10-28T08:47:00+08:00
lastmod: 2022-04-11T08:47:59+08:00
draft: false
tags: ["套件","Visual Studio"]
slug: "visual-studio-ecmascript-level"
# aliases:
#     - /2017/10/visual-studio-ecmascript-level.html
---
## Visual Studio JavaScript 編輯視窗出現 ECMAScript 錯誤提示

因為要要處理前端頁面效果，在參考網路分享的 Vue.js 套件時，一直出現 ECMAScript 錯誤提示，雖然不影響實際功能執行，但編輯畫面上一直出現紅線的錯誤提示不解決還是覺得心裡有疙瘩，所以找了一下解決方式，順手紀錄一下

## 錯誤訊息

* 訊息內容

    ```txt
    ECMAScript 2015 feature. Your current language level is: ECMAScript 5
    ```

* 錯訊截圖

    ![1ecma5](https://user-images.githubusercontent.com/3851540/32129605-12768900-bbbc-11e7-8137-f3c237b2c028.png)

## 解決方式

1. Visual Studio 主選單 Resharper --> Options...

    ![2resharpertools](https://user-images.githubusercontent.com/3851540/32129606-129cbe7c-bbbc-11e7-8442-c490a46a324a.png)

2. Code Editing --> JavaScript --> Inspections --> JavaScript language level

    > 調成所需要的 ECMAScript 版本即可

    ![3scriptlevel](https://user-images.githubusercontent.com/3851540/32129607-12e9bd58-bbbc-11e7-9f16-719c68df6c49.png)

## 心得

原本以為是 Visual Studio 的提示訊息，原來是 Resharper 的，幸虧 Resharper 的相關設定非常彈性，絕大部份需求都可以滿足

## 參考資訊

1. [How can I configure Resharper's language level for ECMAScript 6?](https://stackoverflow.com/questions/32995066/how-can-i-configure-resharpers-language-level-for-ecmascript-6)
