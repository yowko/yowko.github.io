---
title: "如何開啟 Windows Server 2016 遠端桌面"
date: 2017-04-30T09:49:00+08:00
lastmod: 2018-09-18T09:49:27+08:00
draft: false
tags: ["Windows Server 2016"]
slug: "windows-server-2016-enable-remote-desktop"
aliases:
    - /2017/04/windows-server-2016-enable-remote-desktop.html
---
# 如何開啟 Windows Server 2016 遠端桌面
為了體驗 Windows Server 2016 的威力, 從安裝 Windows Server 2016 開始，都是直接操作實體機器，只是這樣不僅常常用錯滑鼠鍵盤，有的軟體授權方式是綁機器的還要準備兩套授權，加上檔案跨機器分享都比較麻煩，所以經過幾次的嘗試後還是決定使用遠端桌面來作業，順手筆記啟用步驟

## 開啟方式 (擇一即可)
1. 使用 GUI 設定
2. 使用 PowerShell

## 使用 GUI 設定

1.  伺服器管理員 --> 杵機伺服器 --> 遠端桌面

    ![1machinemanage](https://cloud.githubusercontent.com/assets/3851540/25560558/fbafa4ec-2d89-11e7-9cff-3a64607c8146.png)

2.  啟用遠端桌面
    *   遠端 --> 允許遠端連線至此電腦

        ![4allow](https://cloud.githubusercontent.com/assets/3851540/25560561/fbf098b2-2d89-11e7-91fc-d73d97aa6a99.png)

    *  會提示防火牆設定跟電源設定

         ![2firewall](https://cloud.githubusercontent.com/assets/3851540/25560559/fbce5572-2d89-11e7-9024-a8fccd17a356.png)

         ![3power](https://cloud.githubusercontent.com/assets/3851540/25560560/fbd74466-2d89-11e7-9618-be52dce1d6d7.png)

## 使用 PowerShell

```ps1
# Enable Remote Desktop
(Get-WmiObject Win32_TerminalServiceSetting -Namespace root\cimv2\TerminalServices).SetAllowTsConnections(1,1) | Out-Null
(Get-WmiObject -Class "Win32_TSGeneralSetting" -Namespace root\cimv2\TerminalServices -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0) | Out-Null
Get-NetFirewallRule -DisplayName "Remote Desktop*" | Set-NetFirewallRule -enabled true
```

# 參考資訊
1.  [如何在 Windows Server 2012 設定遠端桌面連線](https://support.microsoft.com/zh-tw/help/2764220)
2.  [Enable Remote Desktop using PowerShell](http://windowsitpro.com/windows/enable-remote-desktop-using-powershell)
