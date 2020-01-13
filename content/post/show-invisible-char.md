---
title: "如何看到不可視字元"
date: 2020-01-13T23:30:00+08:00
lastmod: 2020-01-13T23:30:31+08:00
draft: false
tags: ["macOS","Tools","Windows 10"]
slug: "show-invisible-char"
---

## 如何看到不可視字元

今天同事遇到個奇異現象：編輯器上看來完全正常的檔案內容，實際執行時卻出現錯誤訊息；刪除提示錯誤位置後重新輸入，git 還真的出現未 commit change ？！

聽到相關描述，立馬讓我聯想偶爾會遇到的問題：不可視字元，過去頭一，兩次遇到這個問題常常都是想破頭、搞得人仰馬翻才解決，之前也曾經想紀錄過，但常常都是發生在十萬火急的時間點，實在無暇保留相關資料來紀錄，今天剛好藉由同事的問題來紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.1
2. Windows 10 1909
3. Visual Studio Code Extension

    - Highlight Bad Chars 0.0.3
    - Gremlins tracker for Visual Studio Code 0.15.1

## 狀況描述

1. syntax error

    ![1syntaxerror](https://user-images.githubusercontent.com/3851540/72267452-0ca0a580-365b-11ea-8a8f-1585ce076bc5.png)

2. git 出現 change

    ![2gitchange](https://user-images.githubusercontent.com/3851540/72267453-0ca0a580-365b-11ea-8c96-20680a7b09bb.png)

## 嘗試解決方式

- 原始內容

    ![4original](https://user-images.githubusercontent.com/3851540/72267457-0d393c00-365b-11ea-801d-758015237914.png)

1. `vi` 編輯器

    > 過去有些特殊字元曾經使用過這個方式解決問題，但這次無效

    `:set list`

    ![3vi](https://user-images.githubusercontent.com/3851540/72267456-0d393c00-365b-11ea-8359-274a5e447e46.png)

2. Windows 環境使用 `Notepad++`

    > 有些特殊字元在 `Notepad++` 中會顯示為方框，或是特殊符號，但本次也無效

    ![2notepadd](https://user-images.githubusercontent.com/3851540/72267455-0ca0a580-365b-11ea-9c67-f08abafe0cf5.png)

3. Visual Studio Code Extension - Highlight Bad Chars

    > 會出現紅色 highlight，提示有特殊字元

    ![5badchar](https://user-images.githubusercontent.com/3851540/72267458-0d393c00-365b-11ea-9e8a-f28c01372fe1.png)

4. Visual Studio Code Extension - Gremlins tracker for Visual Studio Code

    > 行數前面會出現鬼臉，hover 至特殊符號位置會出現對應的 unicode 代碼

    ![Gremlins](https://user-images.githubusercontent.com/3851540/72267459-0d393c00-365b-11ea-80fe-a077b2b3c385.png)

## 心得

過去我在 `vi` 或是 `notepadd++` 就可以看出問題，今天這兩招完全無效，也激起我想找出這個字元的鬥志，雖然 `Visual Studio Code Extension - Highlight Bad Chars` 也可以看出有特殊字元，但一開始的出發點是為了筆記方便而裝的，遇到特殊字視反正就是直接刪掉重 key 就好，直到今天才找到 `Visual Studio Code Extension - Gremlins tracker for Visual Studio Code` 可以直接看到 unicode 更是便利呀，幸虧同事有遇到問題XD

不過同時我也想到如果問題是發生在 linux 環境中，vi 又看不出來，可能又是一陣兵慌馬亂了，我猜最後我可能是哪裡出現 syntax error 就刪哪裡重新輸入吧 ~~ 不知道還有沒有其他方式，請各位大師指教了

## 參考資訊

1. [Highlight Bad Chars](https://marketplace.visualstudio.com/items?itemName=wengerk.highlight-bad-chars)
2. [Gremlins tracker for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=nhoizey.gremlins)
