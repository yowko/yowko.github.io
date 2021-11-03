---
title: "在 macOS 上的 Qt Creator 中出現 failed to parse default search paths from compiler output"
date: 2019-01-27T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["Tools","macOS"]
slug: "failed-to-parse-default-search-paths-from-compiler-output"
---
## 在 macOS 上的 Qt Creator 中出現 failed to parse default search paths from compiler output

這一樣是在 mac 上 build Redis Desktop Manager 時遇到的問題，從之前筆記 [在 macOS 上的 Qt Creator 中出現 No valid kits found](/no-valid-kits-found-on-mac) 先過了第一關後馬來迎來的就是 `failed to parse default search paths from compiler output` 問題，以結果來看來還是 Qt Creator 的設定需要做調整，筆記一下避免日後遇到相同問題又搞不定

## 基本環境說明

1. macOS Mojave 10.14.2
2. Qt 5.12.0
3. Qt Creator 4.8.1

## 錯誤訊息

1. 訊息內容

    ```log
    Could not find qmake spec 'default'.
    Error while parsing file /Users/yowko/opensource/rdm/src/rdm.pro. Giving up.
    Project ERROR: failed to parse default search paths from compiler output
    Error while parsing file /Users/yowko/opensource/rdm/src/rdm.pro. Giving up.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/51801887-8d199b00-227e-11e9-884b-aae17820e69f.png)

## 解決方式

1. 確認問題

    > 檢查在 [在 macOS 上的 Qt Creator 中出現 No valid kits found](/no-valid-kits-found-on-mac) 中為了解決 no valid kits found 問題而手動加入的 Qt 版本的詳細內容

    - QMAKE_SPEC 是 macx-clang

        ![2qmakedetail](https://user-images.githubusercontent.com/3851540/51801888-8d199b00-227e-11e9-8227-7fb629df0546.png)
    - kits compiler 使用 GCC
  
        ![3compilergcc](https://user-images.githubusercontent.com/3851540/51801889-8db23180-227e-11e9-87ab-513687671b79.png)

2. 將 kits 的 compiler 改為 Clang

    ![4useclang](https://user-images.githubusercontent.com/3851540/51801890-8db23180-227e-11e9-81f6-9742a1c641e4.png)

3. 修改前後比較
   - 修改前：使用 GCC

        ![3compilergcc](https://user-images.githubusercontent.com/3851540/51801889-8db23180-227e-11e9-87ab-513687671b79.png)
   - 修改後：使用 Clang

        ![4useclang](https://user-images.githubusercontent.com/3851540/51801890-8db23180-227e-11e9-81f6-9742a1c641e4.png)

## 心得

這個問題我相信對於習慣在 mac 上的開發者應該得花點時間才能找出問題及解決方式，感覺上比較像是 Qt 的使用方式，雖然現在跨平台的 framework 不少像是 Qt、Electron、NW.js...，但終究還沒有一套可以稱王，各套工具還是有各自的擁護者，只能希望這些工具可以愈做愈好，在未來能可以更方便的完成建置

## 參考資訊

1. [在 macOS 上的 Qt Creator 中出現 No valid kits found](/no-valid-kits-found-on-mac)
2. [Mac下用brew配置QT開發環境](http://www.tiger2doudou.com/blog/post/metorm/Mac%E4%B8%8B%E7%94%A8brew%E9%85%8D%E7%BD%AEQT%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)
