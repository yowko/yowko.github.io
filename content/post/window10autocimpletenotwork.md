---
title: "Windows 10 中的 Command Prompt (CMD) 的 Auto-Complete (自動完成) 失效"
date: 2018-10-28T00:42:34+08:00
lastmod: 2021-10-08T00:42:34+08:00
draft: false
tags: ["Windows 10"]
slug: "windows10-enable-autocomplete"
---
## Windows 10 中的 Command Prompt (CMD) 的 Auto-Complete (自動完成) 失效

工作用的電腦系統自從改為 Windows 10 後，原以為過去 Windows 7 與 Windows 10 快捷鍵不同的情況解決後會開始過著幸福美滿的日子，只是果然不能過於樂觀，儘管排除了快捷鍵問題但環境設定不一致問題還是相當嚴重呀，造成許多操作還是東卡西卡，雖然實際動作的延遲時間非常短，但對工作順暢度跟情緒有著不小的影響，思考之後還是決定找方法來徹底解決

今天紀錄的是在 Windows 10 中 Command Prompt (CMD) Auto-Complete (自動完成) 功能失效問題，而這個狀況在自己的電腦上並沒有遇到，但公司電腦卻持續發生，就來看看如何解決吧

## 問題描述

> 進入某個子資料夾按下 `Tab` 卻只插入空白沒有列出適合的資料夾選項

- 預期出現 `C:\blogger`
- 實際出現 `C:\bl   `

    ![1notwork](https://user-images.githubusercontent.com/3851540/47618530-58773580-db0f-11e8-8263-95ce203ee92c.png)

## 調整方式

1. 手動修改註冊機碼
    - 開啟註冊表編輯器

        ![2regedit](https://user-images.githubusercontent.com/3851540/47618531-590fcc00-db0f-11e8-8679-2166fd524482.png)

    - 找到 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Command Processor`

        > 如果不想影響該機器上的其他人設定可以使用 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Command Processor`
    - 將 `CompletionChar` 與 `PathCompletionChar` 值改為 `9`
        - `CompletionChar` 設定資料夾自動完成的觸發鍵
        - `PathCompletionChar` 設定檔案名稱自動完成的觸發鍵

        ![3completionchar](https://user-images.githubusercontent.com/3851540/47618532-590fcc00-db0f-11e8-9e97-0e07405df40e.png)

        ![4changeto9](https://user-images.githubusercontent.com/3851540/47618533-590fcc00-db0f-11e8-8c11-7586ae5efe64.png)

    - 自動完成觸發鍵設定值

        設定值|按鍵
        ---|---
        0x4 |CTRL + D
        0x6 |CTRL + F
        0x9 |TAB

2. 使用 PowerShell
    - 語法

        ```ps1
        Set-ItemProperty -Path "hklm:\SOFTWARE\Microsoft\Command Processor" -Name CompletionChar -Value 9

        Set-ItemProperty -Path "hklm:\SOFTWARE\Microsoft\Command Processor" -Name PathCompletionChar -Value 9
        ```

    - 從 cmd 下執行
        - 至 [EnableWindowsAutoCompletion](https://github.com/yowko/EnableWindowsAutoCompletion) 下載 ps1
        - 執行指令

            ```cmd
            powershell.exe -ExecutionPolicy Bypass -File ".\enableautocomplete.ps1"
            ```

## 心得

這似乎是安裝那麼多次 Windows 以來，第一次遇到自動完成失效的，不知道我又在什麼情況下調到了什麼設定XD

不過也多虧了這次機會多認識了一下原來自動完成的功能可以接受其他按鍵設定，滿有趣的

## 參考資訊

1. [How to Turn On Auto-Complete in the Command Prompt](https://www.online-tech-tips.com/computer-tips/how-to-turn-on-auto-complete-in-the-command-prompt/)
