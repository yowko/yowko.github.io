---
title: "如何修改 Visual Studio Code Terminal 使用程式"
date: 2017-11-12T19:53:00+08:00
lastmod: 2018-09-28T19:53:50+08:00
draft: false
tags: ["Tools"]
slug: "visual-studio-code-terminal"
aliases:
    - /2017/11/visual-studio-code-terminal.html
---
# 如何修改 Visual Studio Code Terminal 使用程式
Visual Studio Code 在啟動速度及跨平台支援上非常優秀，更新速度以及套件豐富性也非常好，因此大受開發人員喜愛，而我大多撰寫 ASP.NET MVC 平常還是使用 Visual Studio 為主，只有想要測試前端框架時才會改用 Visual Studio Code，畢竟速度跟支援度相當有感

之前沒有特別留意過 Visual Studio Code Terminal 一直認為是 cmd，今天在家開啟時才發現是 PowerShell ，想說為什麼更新之後一切變得不同了，於是查了一下原因：`Windows 10 預設使用 PowerShell，之前 OS 則是使用 cmd`，才發現因為公司電腦是 Windows 7 所以是 cmd，自己電腦是 Windows 10 用的就是 PowerShell

只是平常 cmd 用得好好的，不想刻意去查 PowerShell 指令，所以就來看看可以怎麼修改 Visual Studio Code Terminal 使用的程式吧

## 現況

1.  Windows 7

    ![0win7](https://user-images.githubusercontent.com/3851540/32698326-be4f790c-c7de-11e7-85af-eeea9420943f.png)

2.  Windows 10

    ![0win10](https://user-images.githubusercontent.com/3851540/32698327-be7dcc58-c7de-11e7-8d64-dd403dbd403a.png)

## 修改 Terminal 使用程式

1.  開啟 Visual Studio Code 設定

    *   File --> --> Perferences --> Settings

        ![1settings](https://user-images.githubusercontent.com/3851540/32698328-beaa67fe-c7de-11e7-82b3-88c992902c7f.png)

2.  設定 Terminal 使用程式

    *   以下是常見的 shell 執行程式及檔案位置

        ```
        // 預設使用 64-bit cmd ，如果無法使用 64-bit 則改用 32-bit
        "terminal.integrated.shell.windows": "C:\\Windows\\sysnative\\cmd.exe"
        // 預設使用 64-bit PowerShell ，如果無法使用 64-bit 則改用 32-bit
        "terminal.integrated.shell.windows": "C:\\Windows\\sysnative\\WindowsPowerShell\\v1.0\\powershell.exe"
        // Git Bash
        "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe"
        // Bash on Ubuntu (Windows 版)
        "terminal.integrated.shell.windows": "C:\\Windows\\sysnative\\bash.exe"
        ```

3.  重新啟動 Visual Studio Code

    ![2result](https://user-images.githubusercontent.com/3851540/32698329-beda5f68-c7de-11e7-9c11-4149d239d52b.png)

*   需要留意無法同時設定多個不同程式，以最後的設定值為準


## 心得

Visual Studio Code 真心覺得是套好工具，很多需求都被考慮到，很多功能都可以透過設定來調整，真不愧是套深得開發者喜愛的好工具呀

很酷的是官方建議的程式位置是虛擬的，實際上沒那個資料夾，不知道是怎麼做到的

# 參考資訊

1.  [Integrated Terminal](https://code.visualstudio.com/docs/editor/integrated-terminal)
