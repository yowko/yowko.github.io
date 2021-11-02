---
title: "如何消除 IE 中 input 元件上的游標"
date: 2017-01-25T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Frontend","Html"]
slug: "hide-cursor-on-ie-input"
aliases:
    - /2017/01/hide-cursor-on-ie-input.html
---
## 如何消除 IE 中 input 元件上的游標

之前為了解決 HTML 檔案上傳元件在各個瀏覽器呈現效果不同的問題，所以手刻了上傳元件，詳細內容請參考 [試著將不同瀏覽器的上傳元件(input type='file')樣式統一](/customize-file-input-style), 功能面沒問題，但公司認真的 QA 發現在 IE 上出現畫面不一致的狀況，所以我又來了手工調整了 ~~

## 畫面問題

- 實際效果請參考 [原始 jsbin](http://jsbin.com/nufatehize/1/edit)
- `Select` 出現輸入位置提示用的 `|` 游標

    ![IEcursor](https://cloud.githubusercontent.com/assets/3851540/22191301/6a81af22-e164-11e6-8e8f-91b2fa9abbfe.png)

## 解決方式

- 直接在 input 加上 `onfocus="this.blur()"`
  - 原始 input

        ```html
        <input id='file' type="file" name="file" class="x-form-text x-form-field inputtext {required:true}" size="40"/>
        ```
  - 修改後 input

        ```html
        <input id='file' type="file" name="file" onfocus="this.blur()" class="x-form-text x-form-field inputtext {required:true}" size="40"/>
        ```

- 實際效果請參考 [修改後 jsbin](http://jsbin.com/conayezado/1/edit)

## 參考資料

1. [html - Hide textfield blinking cursor - Stack Overflow](https://recalll.co/app/?q=html%20-%20Hide%20textfield%20blinking%20cursor)
2. [試著將不同瀏覽器的上傳元件(input type='file')樣式統一](/customize-file-input-style)
