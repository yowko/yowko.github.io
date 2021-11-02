---
title: "net use 指令出現 error 2148073478"
date: 2017-08-07T22:23:00+08:00
lastmod: 2021-11-02T22:23:45+08:00
draft: false
tags: ["Debug","Jenkins","PowerShell","robocopy"]
slug: "net-use-error-2148073478"
aliases:
    - /2017/08/net-use-error-2148073478.html
---
## net use 指令出現 error 2148073478

同事反應 Jenkins 沒辦法順利將 CI build 的檔案部署至遠端，確認過資料夾權限、robocopy 語法一切正常後，發現原來是 `net use` 指令的權限問題，就來看看問題內容與解決方式吧

## Jenkins 錯誤訊息

* 訊息內容

    ```log
    2017/08/07 17:18:01 ERROR -2146893818 (0x80090006) Creating Destination Directory \\192.168.31.31\D$\

    Invalid Signature.
    ```

* 錯誤截圖

    ![2jenkinserror](https://user-images.githubusercontent.com/3851540/29030711-9a9858ec-7bbe-11e7-8a1e-b7937b15c77f.png)

## 錯誤訊息

* 訊息內容

    ```log
    System error 2148073478 has occurred.
    ```

* 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/29030710-9a930f36-7bbe-11e7-8be9-95f63cf04f57.png)

## 問題發生原因

1. 特定環境
    * Windows Server 2012
    * Windows 8

2. 發生原因細節

    > 問題是由 Windows Server 2012 與 Windows 8 新增至 SMB 3.0 的 "Secure Negotiate" 功能會讓 SMB v2 servers 回應錯誤

## 解決方式

1. 啟用 file server 的簽章 ( PowerShell 語法)

    ```ps1
    Set-SmbClientConfiguration -RequireSecuritySignature $true
    ```

    > 我沒有 file server 試不出這個指令的效果

2. 關閉 client 端的 "Secure Negotiate"

    * <span style="color:red">微軟官方不建議這個做法</span>，可能會有其他資安風險
    * PowerShell 語法

        ```ps1
        Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" RequireSecureNegotiate -Value 0 -Force
        ```

    * 其他注意事項

        > 我設定兩台電腦，有一台需要重啟才生效，有一台不用

    * 設定完成

        ![3success](https://user-images.githubusercontent.com/3851540/29030712-9aa02bf8-7bbe-11e7-892c-6515488b2aba.png)

## 心得

好一陣子沒有 server 的管理權限，設定上也生疏不少，這個問題印象中也遇到，但總是回想不起來怎麼解決，最後還是查了資料才找到解法

另外也想不通為什麼設定會不見，不知道是不是更新造成的；雖然花了一點時間重新設定，但也藉機紀錄一下

## 參考資訊

1. ["System error 2148073478," "extended error," or "Invalid Signature" error message on SMB connections in Windows Server 2012 or Windows 8](https://support.microsoft.com/zh-tw/help/2686098/-system-error-2148073478--extended-error--or-invalid-signature-error-m)
2. [Unable to Map Drives from Windows 8 and Server 2012](https://www.adamfowlerit.com/2013/05/unable-to-map-drives-from-windows-8/)
