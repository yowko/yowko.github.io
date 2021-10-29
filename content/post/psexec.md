---
title: "PsExec - 對遠端執行指令工具的用法"
date: 2017-03-18T02:43:34+08:00
lastmod: 2021-10-28T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "psexec"
aliases:
    - /2017/03/psexec.html
---
## PsExec - 對遠端執行指令工具的用法

原本都是透過 PowerShell 來對遠端電腦執行指令，但不想要將執行的 exe 檔自行複製至遠端電腦，所以查到 PsExec 這個工具支援複製至遠端執行的功能，所以看了該如何使用，順手筆記一下

## 用法

```cmd
psexec [\\ computer [，computer2 [，...] | @file]] [ - u user [-p psswd] [ - ns] [ - r servicename] [ - h] [ - l] [ - s | -e] c [-f | -v]] [ - w directory] ​​[ - d] [ - <priority>] [ - an，n，...]
```

## 參數說明

參數|說明
:---|:---
-a|    指定特定處理器執行應用程式 (For example, 指定 cpu 2 及 cpu 4 來執行程式 : "-a 2,4")
-c|    將指定執行的程式複製至目標電腦上執行;如果未使用這個參數，需確保程式存在目標電腦上的環境變數的 system path 中
-d|    不等執行程序終止 (非互動性).
-e|    不載入特定帳戶的個人資料.
-f|    目標電腦上已存在要執行的檔案，仍然複製
-i|    在目標電腦上建立專屬的 session 來執行程式.未指定會在 console session 下執行
-h|    如果目標系統是 Vista 之後，在允許的情況下會提高帳戶安全性層級來執行程式
-l|    以受限使用者來執行（取消管理者群組權限，只允許指定使用者群組的權限）。  Vista 會使用低完整性來執行
-n|    指定連線至目標電腦連線逾期的時間(單位：秒)
-p|    指定帳號的密碼.如果未使用這個參數提供，會跳出視窗要求密碼
-r|    指定目標電腦上的 service  --> 這我測不出用途
-s|    在目標電腦的系統帳戶下執行程式
-u|    指定登入目標電腦的使用者帳號
-v|    需執行的檔案比目標電腦新才複製檔案
-w|    設定執行的工作目錄 (對於目標電腦而言). --> 這我測不出用途
-x|    Display the UI on the Winlogon secure desktop (local system only).
-priority|    指定不同的優先程式來執行程式： `-low`, `-belownormal`, `-abovenormal`, `-high` or `-realtime`. Vista 可以使用 `-background` 來降低 memory 跟 I/O .
computer|指定 PsExec 執行指令的目標電腦.如果未指定電腦名稱就會直接在本機執行; 如果使用 (\\*) 來指定執行目標，發動電腦及目標電腦需在同一個網域中
@file|執行指令的目標對象清單檔案
cmd|想要執行的應用程式名稱
arguments|執行應用程式的參數(檔案路徑需是目標電腦上的絕對路行).
-accepteula|隱藏授權視窗

## 心得

工具使用上滿方便的，PowerShell 之外的另個選擇，可以列入好用工具清單

## 參考資料

1. [PsExec v2.11](https://technet.microsoft.com/en-us/sysinternals/bb897553.aspx)
