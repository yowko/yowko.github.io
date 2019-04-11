---
title: "將 Windows 10 Quick Link 選單中的 PowerShell 改為 Command Prompt"
date: 2019-02-02T23:45:00+08:00
lastmod: 2019-02-02T23:44:30+08:00
draft: false
tags: ["Windows 10"]
slug: "windows10-replace-powershell-with-cmd"
---
# 將 Windows 10 Quick Link 選單中的 PowerShell 改為 Command Prompt
忘記從什麼時候開始，Windows 10 已經預設將 Quick Link menu( Windows + X) 中的指令工具從 Command Prompt 改為 PowerShell，雖然 PowerShell 功能強大不過日常使用上多數情況下 Commane Prompt 已經非常夠用，雖然印象中過去語法在 cmd 與 PowerShell 有些不太一樣，但最近使用上好像沒有遇到語法相容性問題，想換成 cmd 主要可能還是習慣問題，順手紀錄一下加深印象，實際上用 PowerShell 或 cmd 則又是另外一回事XD

## 基本環境
1. Windows 10 Version 1803 (OS Build 17134.523)
2. 預設 Quick Link 選單 - 使用 PowerShell

    ![1originalquicklink](https://user-images.githubusercontent.com/3851540/52173842-d27e2100-27c6-11e9-804b-c01878a2dc3b.png)

## 修改方式
1. 開啟 Window 選單 (點擊 Windows 圖示 or 按下 Windows key) --> 選擇 `Settings`

    ![2settings](https://user-images.githubusercontent.com/3851540/52173843-d27e2100-27c6-11e9-93c3-e1a8849ca992.png)
2. `Personalization`

    ![3personalization](https://user-images.githubusercontent.com/3851540/52173844-d27e2100-27c6-11e9-91aa-3f6905b7fd10.png)
3. `Taskbar` --> 取消勾選 `Replace Command Prompt with Windows PowerShell in the menu when I right-click the start button or press Windows key+X`

    ![4cancelreplace](https://user-images.githubusercontent.com/3851540/52173845-d316b780-27c6-11e9-94c3-e5b210aca60d.png)

4. 實際效果

    ![5newquickmenu](https://user-images.githubusercontent.com/3851540/52173846-d316b780-27c6-11e9-9b77-3e4f23b1b032.png)

## 心得
Windows 10 近年來更新頻率與過去相比實在快速許多，雖然近幾次穩定性問題被不少使用者垢病，但快速針對使用者回饋做出反應的方式更能貼近使用者實際需求，這次修改 Quick Link 選單即是如此：印象中我之前想要將 Quick Link 選單中的 PowerShell 改為 Command Prompt 得要透過修改 registry 才行，現在只要一個取消勾選的動作，實在方便許多
