---
title: "PowerShell ISE 執行外部執行檔(.exe) 時出現 RemoteException NativeCommandError"
date: 2017-03-22T02:44:34+08:00
lastmod: 2018-09-14T00:42:34+08:00
draft: false
tags: ["PowerShell"]
slug: "powershell-ise-nativecommanderror"
aliases:
    - /2017/03/powershell-ise-nativecommanderror.html
---
# PowerShell ISE 執行外部執行檔(.exe) 時出現 RemoteException NativeCommandError 
打算使用 PowerShell 叫用 PsExec.exe 來執行一些指令，使用 PowerShell ISE 開發時持續出現錯誤訊息，查了些資料，紀錄一下

## 結論
依以往的範例 - debug 就該先說結論：這個是 PowerShell ISE 的內部機制，透過 console 不會出現錯誤提示


## 實際發生現象

>嘗試執行 `nslookup TW.yahoo.com`

1. 以 PowerShell ISE 執行
    
    * 錯誤訊息
        
    ```
    nslookup : Non-authoritative answer:
    At line:1 char:1
    + nslookup TW.yahoo.com
    + ~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : NotSpecified: (Non-authoritative answer::String) [], RemoteException
        + FullyQualifiedErrorId : NativeCommandError
    ```
    - 錯誤截圖
        
        ![1iseerror](https://cloud.githubusercontent.com/assets/3851540/24092603/363f41e6-0d8a-11e7-91f5-08b8b5b2248b.png)
    - 以結果來看有正確回應
2. 以 PowerShell console 執行
    - 正確執行，並正確回應結果
        
        ![2consolepass](https://cloud.githubusercontent.com/assets/3851540/24092601/361cb7f2-0d8a-11e7-8e0e-c8770022233c.png)

## 問題發生原因

>PowerShell ISE 與 PowerShell console 在處理 STDERR (standard error) 時兩者行為不同造成的

外部程式會透過 STDERR (standard error) 來輸出執行資訊，PowerShell console 執行這類外部程式時會另起個 console 來處理程式的回應訊息而繞過了 PowerShell 的訊息處理機制，所以 PowerShell console 本身並不知道 STDERR (standard error) 的存在也就無法揭露錯誤，而 PowerShell ISE 因為沒有另起個 console 來處理，相關訊息會直接讓 PowerShell ISE 處理，以致 STDERR (standard error) 就會被 PowerShell ISE 攔截並標記成錯誤了

## 如何解決
1. 直接使用 PowerShell console
2. PowerShell ISE 中加上 `2>$null`
    > 把 stderr 吃掉
    - `nslookup TW.yahoo.com 2>$null`
        
        ![3resolved](https://cloud.githubusercontent.com/assets/3851540/24092601/361cb7f2-0d8a-11e7-8e0e-c8770022233c.png)

## 心得
這個問題我查了好久，很多網友認為這是 PowerShell 的 bug，也有不少認為是 PowerShell 的設計缺陷。站在使用者的立場，工具的設計方向當然不是我能決定跟批評的，但至少也該有個比較官方的說明文件或是解決方式，不然只是讓使用者更無所適從，使用者遇到問題解決不了無形中也限制了軟體的發展性。有感而發，當然是因為問題查了好幾天還不確定結論是不是正確XD


# 參考資料
1. [PsExec Throws Error Messages, but works without any problems](http://stackoverflow.com/questions/18380227/psexec-throws-error-messages-but-works-without-any-problems)
2. [Error when calling 3rd party executable from Powershell when using an IDE](http://stackoverflow.com/questions/2095088/error-when-calling-3rd-party-executable-from-powershell-when-using-an-ide)
3. [Ignoring an errorlevel != 0 in Windows Powershell](http://stackoverflow.com/questions/1394084/ignoring-an-errorlevel-0-in-windows-powershell/1416933)
4. [Error when calling 3rd party executable from Powershell when using an IDE](http://stackoverflow.com/questions/2095088/error-when-calling-3rd-party-executable-from-powershell-when-using-an-ide)