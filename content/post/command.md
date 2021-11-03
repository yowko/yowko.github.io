---
title: "如何在 Command 中依序執行指令"
date: 2017-07-03T23:31:00+08:00
lastmod: 2021-11-03T23:31:59+08:00
draft: false
tags: ["Tools"]
slug: "command"
aliases:
    - /2017/07/command.html
---
## 如何在 Command 中依序執行指令

同事問說他有幾個 .bat 指令檔，想要上一個 .bat 完成再開始執行下一個 .bat。這樣的需求在 powershell 中很常見，第一個念頭就是問看看能不能用 powershell 寫，但定神細想，難道只能用 powershell 嗎？！ 也許網路上的大神們有其他更漂亮的解法，就來看看可以怎麼做吧

## 基本環境

1. 第一隻 bat：1.bat

    * 印出 `start1`
    * sleep 5 秒
    * 印出 `end1`

    ```cmd
    echo start1
    timeout /t 5
    echo end1
    ```

2. 第一隻 bat：2.bat
    * 印出 `start2`
    * sleep 5 秒
    * 印出 `end2`

    ```cmd
    echo start2
    timeout /t 5
    echo end2
    ```

## 讓 command 循序執行

* 使用 `&&` 串接指令

    ```cmd
    1.bat && 2.bat
    ```

* 效果
    1. 執行 `1.bat` 並等待倒數結數

        ![11bat](https://user-images.githubusercontent.com/3851540/27799379-8294dd48-6047-11e7-879a-3959743fde77.png)

    2. `1.bat` 完成後執行 `2.bat` 並等待倒數結數

        ![22bat](https://user-images.githubusercontent.com/3851540/27799380-82b9466a-6047-11e7-8601-5f32eb913141.png)

    3. 指令全部完成

        ![3done](https://user-images.githubusercontent.com/3851540/27799378-8265cb7a-6047-11e7-98b4-00d44c3bf54f.png)

## 其他 command 好用符號指令

|符號|說明|範例|
|--- |--- |--- |
|`>`|將指令執行結果寫至檔案;<br/>檔案不存在時會自動建立;<br/>如果檔案已存在會直接覆寫|echo YowkoTest > test.txt|
|`>>`|將指令執行結果加至檔案末端;<br/>檔案不存在時會自動建立|echo YowkoTestAppend >> test.txt|
|`<`|將檔案內容當做指令輸入來源|more < test.txt|
|`¦`|將指令 1 執行結果當做指令 2 輸入參數|echo y`¦`del test.txt|
|`&`|將兩個指令合併執行，先執行指令 1 接著執行指令 2|DIR Z:\ & 2.bat (z 不存在，2.bat 仍會執行)|
|`&&`|合併執行，指令 1 執行成功後接著執行指令 2|DIR Z:\ && 2.bat(z 不存在，2.bat 不會執行)|
|`¦¦`|只有在指令 1 執行失敗時才執行指令 2|DIR Z:\ `¦¦` 2.bat ((z 不存在，2.bat 才會執行)|
|`@`|通常放在 bat 檔的第一行，用來關閉顯示指令|@echo off|

## 心得

雖然現在比較常使用 PowerShell，不過 command 指令通用性還是比較高，語法的多樣性或許比不上 PowerShell，如果懂得充份利用還是可以完成很多事的

## 參考資訊

1. [DOS is dead, long live the command line](https://commandwindows.com/command1.htm)
2. [如何在批次檔(Batch)中實現 sleep 命令讓任務暫停執行 n 秒](http://blog.miniasp.com/post/2009/06/24/Sleep-command-in-Batch.aspx)
3. [Windows Command Prompt in 15 Minutes](http://www.cs.princeton.edu/courses/archive/spr05/cos126/cmd-prompt.html)
4. [Redirection](https://ss64.com/nt/syntax-redirection.html)
