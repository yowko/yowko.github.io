---
title: "取得 Word(.docx) 中的內嵌檔案"
date: 2018-01-17T01:22:00+08:00
lastmod: 2020-12-11T01:22:40+08:00
draft: false
tags: ["Office"]
slug: "extract-embedded-file-in-word"
aliases:
    - /2018/01/extract-embedded-file-in-word.html
---
# 取得 Word(.docx) 中的內嵌檔案
之前筆記 [使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF](/2018/01/c-sharp-word-to-pdf.html) 曾經分享過如何使用 C# 將 Word 轉成 PDF，同事在實作後 user 反應如果 word 中有其他文件(e.x. pdf、word、excel、zip) 並沒有同步進行轉換，造成資料遺失

不是我刻意想吐糟 user 反應沒有處理 word 中有其他文件的問題(user 永遠是對的XD)，但在 user 提出需求前根本就沒有說 word 會內嵌其他文件，根據最小可行產品的原則一定是沒做的呀 哈哈

今天就先紀錄一下不使用程式下的做法

## 前提說明

> Word 檔中內嵌 excel 為例

*   插入 --> 物件

    ![1insert](https://user-images.githubusercontent.com/3851540/35001985-24060082-fb23-11e7-8ab5-501b96f2f685.png)

*   檔案來源 --> 瀏覽 (選擇檔案) --> 以圖示顯示

    ![2icon](https://user-images.githubusercontent.com/3851540/35001987-243f9a7c-fb23-11e7-92c0-68cf10f55e78.png)

*   文件內容嵌入 excel

    ![3example](https://user-images.githubusercontent.com/3851540/35001989-249a9314-fb23-11e7-9e7d-2a5fa4be118a.png)

## 取得嵌入檔案

1.  方法一(適合單檔)：直接點擊開啟檔案後另存新檔
2.  方法二(適合多檔)：
    *   將 word 附檔名改為 `.zip`

        ![4convert2zip](https://user-images.githubusercontent.com/3851540/35001990-24d28dfa-fb23-11e7-984b-206b9d41aa31.png)

    *   使用解壓縮工具開啟該 zip 檔

        ![5openzip](https://user-images.githubusercontent.com/3851540/35001991-2516f544-fb23-11e7-967c-d5d2c42e9356.png)

    *   `word\embeddings` 存放著所有嵌入檔案

        ![6embedded](https://user-images.githubusercontent.com/3851540/35001993-2646ae6e-fb23-11e7-8a37-7e152db2a910.png)

## 心得

今天筆記內容完全沒有提到程式實作，主要是以前沒有用過`圖示顯示`在 word 中嵌入其他物件，只有將 excel 表格嵌入的經驗，所以特別紀錄一下用法

除此之外也紀錄如何在沒有程式碼協助取出內嵌檔案的簡易做法，知道做法後就可以透過程式讓整個流程自動化

# 參考資訊

1.  [How to Extract Images, Text, and Embedded Files from Word, Excel, and PowerPoint Documents](https://www.howtogeek.com/50628/easily-extract-images-text-and-embedded-files-from-an-office-2007-document/)
