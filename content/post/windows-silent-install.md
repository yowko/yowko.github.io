---
title: "如何在 windows 環境中使用指令來安裝程式"
date: 2017-04-29T10:28:00+08:00
lastmod: 2020-04-16T10:29:14+08:00
draft: false
tags: ["Tools","Windows"]
slug: "windows-silent-install"
aliases:
    - /2017/04/windows-silent-install.html
---
## 如何在 windows 環境中使用指令來安裝程式

保哥在 Windows container 的課堂上考了我個問題：如果沒有 GUI 介面要如何安裝程式(e.g.更新 .NET Framework)，正確答案是透過標準的 msi 檔安裝，而我只想到用 powershell 跟 chocolatey 安裝，查了資料發現實際上背景還是靠著 msiexec 的靜默模式

既然不懂就來了解一下其中的用法吧

## 參數說明

|Option|Parameters|Meaning|
|--- |--- |--- |
|/I|Package｜ProductCode|安裝或是設定程式|
|/f|[p｜o｜e｜d｜c｜a｜u｜m｜s｜v] Package｜ProductCode|執行修復動作. 這個選項會導致 command line 輸入的屬性值失效.預設是參數是 `omus`，詳細資訊可以參考 [REINSTALLMODE](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa371182%28v=vs.85%29.aspx)<br/>p - 只有在檔案遺失時重新安裝  <br/>o - 檔案遺失或是已安裝舊版本時重新安裝  <br/>e - 檔案遺失、已安裝相同或是舊本時重新安裝  <br/>d - 檔案遺失或是已安裝其他版本時重新安裝  <br/>c - 檔案遺失或檔案驗證碼不相符時重新安裝.如果檔案中有 msidbFileAttributesChecksum 這個 attribute 時僅修復檔案  <br/>a - 強制重新安裝所有檔案.  <br/>u - 重寫所有必要的 user 特定機碼.  <br/>m - 重寫所有必要的 computer 特定機碼.  <br/>s - 覆寫所有已存在的捷徑.  <br/>v - 從源頭執行並且重新 cahce local 組件.首次安裝時不要使用 `v` 選項|
|/a|Package|從網路安裝,詳細資訊可以看 [Administrative installation](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa367541%28v=vs.85%29.aspx)(這我看不懂意思)|
|/x|Package｜ProductCode|解除安裝|
|/j|[u｜m]Package　or  <br/>[u｜m]Package/t Transform List or[u｜m]Package/g LanguageID|程式通知 這個選項將會忽略其他 command line 輸入的屬性值. <br/>u - 通知現有 user.  <br/>m - 通知所有 user.  <br/>g - 語言指標.  <br/>t - 將轉換套用至已通知的組件|
|/L|[i｜w｜e｜a｜r｜u｜c｜m｜o｜p｜v｜x｜+｜!｜*] Logfile|將 log 紀錄至指定位置.安裝程式不會建立資料夾，logfile 位置必需已經存在的. Flags 會指定該 log 哪些資訊. 如果沒有 flags 預設值是 'iwearmo.'  <br/>i - 狀態訊息.  <br/>w - 非嚴重警告.  <br/>e - 所有錯誤訊息.  <br/>a - 啟動操作.  <br/>r - 特定行為紀錄.  <br/>u - 使用者要求.  <br/>c - 初始化 UI 參數.  <br/>m - 記憶體不足 or 嚴重退出資訊.  <br/>o - 磁碟空間不足訊息.  <br/>p - 終端機的屬性.  <br/>v - 詳細輸出.  <br/>x - 額外的 debug 資訊.  <br/>Windows Installer 2.0:  不支援. Windows Installer version 3.0.3790.2180 之後才支援.  <br/>+ - 加入至已存在的檔案.  <br/>! - 輸出每一行至 log .  <br/>"*" - 萬用字元, 紀錄所有資訊除了`v` 及 `x` 選項. 要包含 `v` 及 `x` 選項就用`/l*vx`.  <br/>Note：更多設定及方法可以看 [Normal Logging](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa370536%28v=vs.85%29.aspx) 及 [Windows Installer Logging](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa372847%28v=vs.85%29.aspx)|
|/m|filename <br/>Note：檔名長度不得超過 8 .|產生 SMS 狀態 .mif 檔. 必需與 install (-i), remove (-x),administrative installation (-a), or reinstall (-f) 一起使用. ISMIF32.DLL 會被當做 SMS 的一部份來安裝而且必需在路徑上.  <br/>狀態 mif 檔的欄位中包含資訊：:  <br/>Manufacturer - [Author](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa367808%28v=vs.85%29.aspx)<br/>Product - [Revision Number](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa371255%28v=vs.85%29.aspx)<br/>Version - [Subject](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa372033%28v=vs.85%29.aspx)<br/>Locale - [Template](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa372070%28v=vs.85%29.aspx)<br/> Serial Number - not set  <br/>Installation - set by ISMIF32.DLL to "DateTime"  <br/>InstallStatus - "Success" or "Failed"  <br/>Description - 錯誤訊息的順序如下：: 1) 安裝程式產生的錯誤訊息. 2) 來自 Msi.dll 的訊息，如果安裝無法啟動或用戶退出，Msi.dll的資源.3)系統錯誤訊息檔案. 4) 格式化訊息: "Installer error %i", %i 是 Msi.dll 回傳的錯誤.|
|/p|PatchPackage[;patchPackage2…]|提供修補程式.要將修補程序應用於已安裝的管理映像，必須組合下列選項:  `/p[;patchPackage2…] /a `|
|/q|n｜b｜r｜f|設定 [user interface level](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa372391%28v=vs.85%29.aspx).  <br/>q , qn - 沒有 UI  <br/>qb - Basic UI. `qb!` 可以隱藏 Cancel 按鈕.  <br/>qr - Reduced UI 在安裝結束時不顯示 modal 對話框.  <br/>qf - Full UI 在安裝結束時顯示 FatalError, UserExit, or Exit modal 對話視窗.  <br/>qn+ - No UI 除了安裝結束時會顯示 modal 對話窗.  <br/>qb+ - Basic UI 並且在安裝結束時會顯示 modal 對話窗. 但如果是 user 取消就不會顯示對話框. `qb+!` or `qb!+` 可以隱藏 Cancel 按鈕.  <br/>qb- - Basic UI 不顯示 modal 對話視窗. `/qb+-` 不支援. `qb-!` or `qb!-` 可以隱藏 Cancel 按鈕.  <br/>Note `!` 選項在 Windows Installer 2.0 只支援 basic UI. 對 full UI 是無效的.|
|/? or /h||顯示 Windows Installer 的版權資訊.|
|/y|module|在 command line 中呼叫系統函式 DllRegisterServer 來自行註冊 module.指定 DLL 完整路徑. 範例：`msiexec /y c:\yowko.DLL` 這個選項只能用來為無法使用 .msi 檔案註冊表格的程式註冊資訊|
|/z|module|在 command line 中呼叫系統函式 DllUnRegisterServer 來取消註冊. 指定 DLL 完整路徑. 範例：`msiexec /z c:\yowko.DLL`  這個選項只能用來為無法使用 .msi 檔案註冊表格的程式移除資訊|
|/c||宣傳產品的新執行個體. 必需跟`/t`一起使用. 這個選項從 Windows Server 2003 and Windows XP SP1 開始支援.|
|/n|ProductCode|指定特定的產品執行個體.安裝產品多次可以利用產品代碼轉換來識別不同執行個體. 這個選項從 Windows Server 2003 and Windows XP SP1 開始支援.|

## 注意事項

* `/i`, `/x`, `/f[p|o|e|d|c|a|u|m|s|v]`, `/j[u|m]`, `/a`, `/p`, `/y` ,`/z` 不該同時使用

    > 例外：patch 管理安裝會同時使用 `/p`與`/a`

* `/t`, `/c`  `/g` 必需跟 `/j` 一起使用.
* `/l` , `/q` 可以跟 `/i`, `/x`, `/f[p|o|e|d|c|a|u|m|s|v]`, `/j[u|m]`, `/a`, `/p` 一起使用
* `/n` 可以跟 `/i`, `/f`, `/x`,`/p`一起使用.

## 實際使用

* 安裝

    > `msiexec /i A:\Example.msi`

* 指定屬性

    * command line 屬性名稱都會被當做大寫處理，但實際屬性是區分大小寫的，詳細資訊可以參考 [About Properties](https://msdn.microsoft.com/zh-tw/library/windows/desktop/aa367437%28v=vs.85%29.aspx)
    
    * 留意語法順序
        
        > `msiexec /i A:\Example.msi PROPERTY=VALUE`

    * 值有空格，請用`"`包

        > `msiexec /i A:\Example.msi PROPERTY="Embedded White Space"`

    * 清除設定

        > `msiexec /i A:\Example.msi PROPERTY=""`

    * 如果值中有`"`，請再用 `""` 取代

        > `msiexec /i A:\Example.msi PROPERTY="Embedded ""Quotes"" White Space"`

* 通知選項不分大小寫

    > `msiexec /JM msisample.msi /T transform.mst /LIME logfile.txt`

* 支援多個安裝個體轉換

    > `msiexec /JM msisample.msi /T :instance1.mst;customization.mst /c /LIME logfile.txt`

* patch

    > `msiexec /p msipatch.msp;msipatch2.msp /n {00000001-0002-0000-0000-624474736554} /qb`

* `/i` 與 `/p` 無法同時使用的做法

    > `msiexec /i A:\Example.msi PATCH=msipatch.msp;msipatch2.msp /qb`

* `/p` 與 `PATCH` 無法同時存在，`/p`會覆寫 `PATCH`

## 心得

功能很多，尤其 advertise 跟 administrative 相關功能實在看不太懂XD，先紀錄一下以後用到時可以比較快上手就好, 如果有寫錯，請大家指教了

## 參考資訊

1. [Command-Line Options](https://msdn.microsoft.com/en-us/library/windows/desktop/aa367988%28v=vs.85%29.aspx)
