---
title: "如何執行遠端 PowerShell Script"
date: 2016-12-21T00:42:34+08:00
lastmod: 2021-10-28T00:42:34+08:00
draft: false
tags: ["PowerShell"]
slug: "run-remote-powershell"
aliases:
    - /2016/12/Execute-Remote-PowerShell-Script.html
    - /2016/12/run-remote-powershell/
---
## 如何執行遠端 PowerShell Script

百敬老師之前在推廣活動上說，他認為 PowerShell 是近期最值得投資的技能，相信以他的身份地位及眼光，絕對是不會錯的，可惜的是一直沒有機會可以實際應用，最近剛好有專案可以試試，當然要來感受大師推薦語言的魔力。

## 使用方式

1. <span style="color:red"> PowerShell 需以 Administrator 身份執行</span>
2. 確認是否來源端及目標端都已加入 domain
   - 未加入 domain 的錯誤

        ```ps1
        New-PSSession -ComputerName 192.168.31.247
        ```

        ![1domainneed](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1200x251/c8533a46ac2d458e4c1992d642e62a76/_output_1domainneed.png)

   - 連線時加入 Credential

        ```ps1
        New-PSSession -ComputerName 192.168.31.247 -Credential
        ```

3. 啟用目標端電腦的 WinRM 服務

    ```ps1
    Get-Service WinRm
    ```

    - Running

        ![2winrmunning](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/715x118/bdddf20d27a7b522f2259e15e7662292/_output_2winrmrunning.png)

    - Stopped

        ![2servicestop](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/746x121/0b257551639ea53901a6bb1caa7fa93b/_output_2servicestop.png)

    - `Start-Service WinRm` 可以啟用 WinRM Service

4. 允許目標端電腦遠端執行 PowerShell

    - `Enter-PSSession -ComputerName localhost`
        - 可檢查是否允許遠端執行

           ![4disableremoting](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1053x165/b96353cbe6db390182eec2940179c36f/_output_4disableremoteing.png)

    - `Enable-PSRemoting –force`
        - 允許遠端執行

5. 來源端與目標端不在同一個 domain 時

    - 來源端需將目標端加入信任清單(將 遠端 加入 欲發動 PowerShell Script 電腦的信任清單)
    - 需以 `電腦名稱` or `IP` 加入信任清單
       - `winrm s winrm/config/client '@{TrustedHosts="ComputerName"}'`
       - `Set-Item WSMan:\localhost\Client\TrustedHosts "ComputerName"`
    - 重啟 WinRM service
       - `Restart-Service WinRM`

6. 快速設定

    ```ps1
    WinRM quickconfig
    ```

7. 錯誤排除

    - errorcode 0x8009030e
        - 有設定信任清單但未指定 Credential
        - 從 Win 10 連線 Server 2016
        - 英文錯誤訊息

            ![3ox8009090e](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1200x403/331dc79316547acadd6e3e5fd5d659e9/_output_3ox8009090e.png)

        ```ps1
        New-PSSession : [Yowko-Server2016] Connecting to remote server  Yowko-Server2016 failed with the            llowing error  message : WinRM cannot process the request. The following error  with errorcode    009030e    occurred while using Negotiate  authentication:
        A specified logon session does not exist. It may already have  been terminated.
        Possible causes are:
        -The user name or password specified are invalid.
        -Kerberos is used when no authentication method and no user name  are specified.
        -Kerberos accepts domain user names, but not local user names.
        -The Service Principal Name (SPN) for the remote computer name  and port does not exist.
        -The client and remote computers are in different domains and  there is no trust between the two domains.
        After checking for the above issues, try the following:
        -Check the Event Viewer for events related to authentication.
        -Change the authentication method; add the destination computer  to the WinRM TrustedHosts configuration           etting or use  HTT
        PS transport.
        Note that computers in the TrustedHosts list might not be  authenticated.
        -For more information about WinRM configuration, run the  following command: winrm help config. For more            nformation, see the about_Remote_Troubleshooting Help topic.
        At line:1 char:1
        + New-PSSession -ComputerName Yowko-Server2016
        + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],           PSRemotingTran sportException
        + FullyQualifiedErrorId : 1312,PSSessionOpenFailed
        ```

    - Access is denied.
        - Credential 錯誤

            > 從 Win 10 連線 Server 2016

        - 未指定 Credential

            > 從 Server 2016 連線 Win 10

        - 中文錯誤訊息

            ![1accessdeny](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1184x146/cf7e3562636c48943b1ea10240d5d689/_output_1accessdeny.png)

            ```ps1
            New-PSSession : [192.168.31.102] 連線到遠端伺服器 192.168.31. 102 失敗，傳回下列錯誤訊息: 存取被拒。 如需詳細資訊，請參閱  about_Remote_Troubleshooting 說明主題。位於 線路:1 字元:1    +  New-PSSession -ComputerName 192.168.31.102    +  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],  PSRemotingTran        sportException
                + FullyQualifiedErrorId : AccessDenied, PSSessionOpenFailed
            ```

        - 英文錯誤訊息

            ![4accessdenied](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1200x179/f55cbea1f5803d06f334f4c40f8d227d/_output_4accessdenied.png)

        ```ps1
        New-PSSession : [Yowko-Server2016] Connecting to remote server  Yowko-Server2016 failed with the ollowing error message : Access  is denied. For more information, see the  about_Remote_Troubleshooting elp topic.    
        At line:1 char:1    
        + New-PSSession -ComputerName Yowko-Server2016 -Credential  administrato ...    
        +  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ~~~~        
        + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],PSRemotingTran  sportException        
        + FullyQualifiedErrorId : AccessDenied,PSSessionOpenFailed
        ```

    - The WinRM client cannot process the request
        - 未將執行目標加入信任清單
        - 中文錯誤訊息

            ![3htrustedhost](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1175x187/68c3177160b3cbfe9022c506553e1bed/_output_3htrustedhost.png)

        ```ps1
        New-PSSession : [192.168.31.102] 連線到遠端伺服器 192.168.31.102 失 敗，傳回下列錯誤訊息: WinRM 用戶端無法處該要求。若驗證配置與 Kerberos 不 同，或是用戶端電腦沒有加入網域， 則必須使用 HTTPS 傳輸，或是將目標電腦新增 到 rustedHosts 組態設定中。 請使用 winrm.cmd 來設定 TrustedHosts。請注 意，可能不會驗證在 TrustedHosts 清單中的腦。 您可以執行下列命令，以取得相關 的詳細資訊: winrm help config。 如需詳細資訊，請參閱  bout_Remote_Troubleshooting 說明主題。
        位於 線路:1 字元:1    
        + New-PSSession -ComputerName 192.168.31.102 -Credential  "y****** ...    
        +  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ~~~~    
        + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],     PSRemotingTransportException    
        + FullyQualifiedErrorId : ServerNotTrusted,PSSessionOpenFailed
        ```

        - 英文錯誤訊息

            ![5trustedhost](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1200x239/917c1de02dc4db61b0ebe0c2412d0e2b/_output_5trustedhost.png)

        ```ps1
        New-PSSession : [Yowko-Server2016] Connecting to remote server  Yowko-Server2016 failed with the ollowing error message : The  WinRM client cannot process the request. If the authentication  scheme is ifferent from Kerberos, or if the client computer is  not joined to a domain, then HTTPS transport must e used or the  destination machine must be added to the TrustedHosts  configuration setting. Use inrm.cmd to configure TrustedHosts.  Note that computers in the TrustedHosts list might not be  uthenticated. You can get more information about that by running  the following command: winrm help onfig. For more information,  see the about_Remote_Troubleshooting Help topic.
        At line:1 char:1    
        + New-PSSession -ComputerName Yowko-Server2016 -Credential  administrato ...    
        +  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ~~~~        
        + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],PSRemotingTran     sportException        
        + FullyQualifiedErrorId : ServerNotTrusted,PSSessionOpenFailed
        ```

    - WinRM cannot complete the operation.

        > 連線時加上 `-UseSSL`

        - 雖然文件上說某些情況(cross domain) 需要使用 SSL，但預設是不啟用， telnet 預設的 HTTPS port(5986) 是不通的

            ![8httpsport](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/847x335/3e0a32e6d2c5f5ac4983a4c00de6fe8c/_output_8httpsport.png)

        - 中文錯誤訊息

            ![6firewall](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1179x176/545feb9158264647375efb60f5d11f08/_output_6firewall.png)

        ```ps1
        New-PSSession : [YowkoMac-WIN10] 連線到遠端伺服器 YowkoMac-WIN10 失 敗，傳回下列錯誤訊息: WinRM 無法完成作。
        請確認指定的電腦名稱有效、可經由網路連接電腦，而且 WinRM 服務的防火牆例外已 啟用且可從這部電腦存取。
        依預設，公用設定檔的 WinRM 防火牆例外會限制相同本機子網路內對遠端電腦的存 取。 
        如需詳細資訊，請參閱 about_Remote_Troubleshooting 說明主題。
        位於 線路:1 字元:1    
        + New-PSSession -UseSSL -ComputerName YowkoMac-WIN10
        + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
        + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],PSRemotingTran     sportException    
        + FullyQualifiedErrorId : WinRMOperationTimeout, PSSessionOpenFailed
        ```

        - 英文錯誤訊息

            ![7firewall](https://trello-attachments.s3.amazonaws.com/58565d13cc3c5798c4ee263b/1200x241/51c5c70d1cab410c0fda8ec8ad9d34a7/_output_7firewall.png)

        ```ps1
        New-PSSession : [192.168.31.247] Connecting to remote server 192. 168.31.247 failed with the following rror message : WinRM cannot  complete the operation. 
        Verify that the specified computer name is valid, that the  computer is accessible over the network, and hat a firewall  exception for the WinRM service is enabled and allows access  from this computer. 
        By default, the WinRM firewall exception for public profiles  limits access to remote computers within he same local subnet.  For more information, see the about_Remote_Troubleshooting Help  topic.
        At line:1 char:1    
        + New-PSSession -UseSSL -ComputerName 192.168.31.247    
        + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~    
        + CategoryInfo          : OpenError: (System.Manageme.... RemoteRunspace:RemoteRunspace) [New-PSSession],PSRemotingTran     sportException    
        + FullyQualifiedErrorId : WinRMOperationTimeout, PSSessionOpenFailed
        ```

## 心得

PowerShell 不用編譯相當棒，百敬老師也非常推崇，可以直接看到程式碼，可以馬上做調整。

就我自己使用上來看，PowerShell 進入障礙是比較高的，相當於 Microsoft 的 VB 及 C# 而言，學習資料不僅較少也較沒系統性，容易造成卡關。雖然 ISE 開發工具已經非常好用，但被 Visual Studio 慣壞的我  還是覺得 intellisense 效果及提示相當不足。

而實際設定上也容易出現提示不明確與文件難以搜尋的困難。

不過如果做為 Server 管理工具，還是挺方便的。

## 參考資料

1. [如何在遠程伺服器上運行PowerShell命令？](https://read01.com/DQ3M4.html)
2. [執行遠端命令](https://msdn.microsoft.com/zh-tw/powershell/scripting/core-powershell/running-remote-commands)
3. [Tip: Enable and Use Remote Commands in Windows PowerShell](https://technet.microsoft.com/en-us/library/ff700227.aspx)
4. [How to Run PowerShell Commands on Remote Computers](http://www.howtogeek.com/117192/how-to-run-powershell-commands-on-remote-computers/)
