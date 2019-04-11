---
title: "PowerShell 基本語法筆記"
date: 2017-01-12T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["PowerShell"]
slug: "powershell-basic-script"
aliases:
    - /2017/01/powershell-basic-syntex.html
---
# PowerShell 基本語法筆記
最近工作上的需要，用了 PowerShell 語法來處理，幾乎各式各樣的花式需求都可以達成，怪不得百敬老師非常推薦 PowerShell ，功能十分強大呀，只可惜學習資源比較零碎，所以特別紀錄一下最近用到的語法，避免很快忘記又查了老半天XD

## 數字處理
1. 比較大小
    
    ```ps1
    $a -gt $b #($a 是否大於 $b)
    ```

## 字串處理
1. 字串分割
    - 1-1. 
        
        ```ps1
        $$serveritem -split ',' 
        ```
    - 1-2.
        
        ```ps1
        $serveritem.Split(',')
        ```

2. 檢查是否為空值 (string.IsNullOrWhiteSpace)
    
    ```ps1
    [string]::IsNullOrWhiteSpace($serveritem.ExcludeFolder)
    ```

3. 比對字串開頭
     
     ```ps1
     $stringVariable.StartsWith('startString','CurrentCultureIgnoreCase')
     ```

4. 字串比對
     
     ```ps1
     $stringVariable -eq "Value"
     ```


## 檔案處理
1. 檔案內容轉成 json
    
    ```ps1
    Get-Content -Raw -Path $filelocation | ConvertFrom-Json
    ```

2. 檔案壓縮與解壓縮
    - PowerShell v5 
        - 壓縮
            - 語法
                
                ```ps1
                Compress-Archive -Path $sourceFolder -DestinationPath $archiveFile
                ```
            - 實際範例
                
                ```ps1
                Compress-Archive -Path D:\Downloads\Demo -DestinationPath D:\Downloads\demo.zip
                ```
        - 解壓縮
            - 語法
                
                ```ps1
                Expand-Archive -Path $archiveFile -DestinationPath $targetFolder
                ```
            - 實際範例
                
                ```ps1
                Expand-Archive -Path D:\Downloads\demo.zip -DestinationPath D:\Downloads\testPS
                ```
    - other
        - 壓縮
            - 語法
                
                ```ps1
                Add-Type -AssemblyName 'System.IO.Compression.Filesystem'
                [System.IO.Compression.ZipFile]::CreateFromDirectory($folderSource,$fileDestination)
                ```
            - 實際例範
                
                ```ps1
                $folderSource='D:\Downloads\Demo'
                $fileDestination='D:\Downloads\demo.zip'

                Add-Type -AssemblyName 'System.IO.Compression.Filesystem'
                [System.IO.Compression.ZipFile]::CreateFromDirectory($folderSource,$fileDestination)
                ```
        - 解壓
            - 語法
                
                ```ps1
                Add-Type -AssemblyName 'System.IO.Compression.Filesystem'
                [System.IO.Compression.ZipFile]::ExtractToDirectory($fileSource, $folderDestination)
                ```
            - 實際範例
                
                ```ps1
                $fileSource='D:\Downloads\demo.zip'
                $folderDestination='D:\Downloads\testPS'
                Add-Type -AssemblyName 'System.IO.Compression.Filesystem'
                [System.IO.Compression.ZipFile]::ExtractToDirectory($fileSource, $folderDestination)
                ```
                
## List 處理
1. 多個 List 取交集或是聯集
    
    ```ps1
    $a = (1,2,3,4)
    $b = (1,3,4,5)
    ```
    - 1-1. 聯集
        
        ```ps1
        $a + $b | select -uniq    #union
        ```
    - 1-2. 交集
        
        ```ps1
        $a | ?{$b -contains $_}   #intersection
        ```

2. List 加入 item
    - 2-1. 宣告個空的 List
        
        ```ps1
        $a = New-Object System.Collections.ArrayList
        ```
    - 2-2. 加入項目
        
        ```ps1
        $a.Add('data')
        $a.Add('test')
        ```

## Credential 相關
1. 指定 Credential
    
    ```ps1
    #用來儲存密碼(僅需輸入一次，會加密儲存)
    read-host -assecurestring | convertfrom-securestring | out-file C:\securestring.txt 
    #取出密碼(以下兩句效果相同)
    $pass = ConvertTo-SecureString -String (Get-Content C:\securestring.txt)  
    #$pass = cat C:\securestring.txt | convertto-securestring  
    #使用 username 及 密碼  建立 credential
    $Cred = new-object -typename System.Management.Automation.PSCredential -argumentlist "AD\username",$pass    
    ```
2. 取得密碼明碼
    
    ```ps1
    $cred.GetNetworkCredential().Password
    ```

## Windows 服務相關
1. 取得服務狀態
    
    ```ps1
    Get-Service -Name "serviceName" #Get-Service -Name "aspnet_state"
    ```

2. 停用服務
    
    ```ps1
    Stop-Service -Name "aspnet_state"
    ```

3. 啟用服務
    
    ```ps1
    Start-Service -Name "aspnet_state"
    ```

## IIS 相關
1. AppPool
    - 1-1. 取得特定 AppPool 狀態
    
        ```ps1
        Get-WebAppPoolState -Name "appPoolName" 
        ```

    - 1-2. 停用 AppPool
    
        ```ps1
        Stop-WebAppPool -Name "appPoolName"
        ```

    - 1-3. 啟用 AppPool
    
        ```ps1
        Start-WebAppPool -Name "appPoolName"
        ```

2. Website
    - 2-1. 取得特定網站狀態
    
        ```ps1
        Get-WebsiteState -Name "websiteName"
        ```

    - 2-2. 停用特定網站
    
        ```ps1
        Stop-Website -Name "websiteName"
        ```

    - 2-3. 啟用特定網站
        
        ```ps1
        Start-Website -Name "websiteName"
        ```

## 動態組成指令

```ps1
$ExecutionContext.InvokeCommand.NewScriptBlock(" /XD $parameter")
```

## 遠端執行
- Invoke-Command
    - 語法
        
        ```ps1
        Invoke-Command –ComputerName Production1 {$env:COMPUTERNAME}
        ```
    - 傳遞其他參數語法
        
        ```ps1
        Invoke-Command -ComputerName $Computer -ScriptBlock {param($comp) write-host "This script is running on machine: $Comp" } -ArgumentList $Computer
        ```

- New-PSSession

    ```ps1
    New-PSSession –ComputerName Production1
    Enter-PSSession {sessionID}
    ```

## 無線網路服務
1. 確認無線網路服務的狀態
    
    ```ps1
    Get-WindowsFeature *Wireless*
    ```
2. 安裝無線網路服務
    
    ```ps1
    Install-WindowsFeature -Name Wireless-Networking
    ```
3. 啟動服務
    
    ```ps1
    net start WlanSvc
    ```

# 參考資料
1. [passing array of strings into function](https://social.technet.microsoft.com/Forums/windowsserver/en-US/4760846a-3b31-401f-a49e-92e87beeb367/passing-array-of-strings-into-function?forum=winserverPowerShell)
2. [Arrays](http://ss64.com/ps/syntax-arrays.html)
3. [Windows PowerShell: When to Choose Workflow Versus Script Functions](http://www.informit.com/articles/article.aspx?p=2481897)
4. [Creating a Workflow by Using a Windows PowerShell Script](https://msdn.microsoft.com/en-us/library/jj148984%28v=vs.85%29.aspx)
5. [Invoke-Command](https://msdn.microsoft.com/en-us/PowerShell/reference/5.1/microsoft.PowerShell.core/invoke-command)
6. [POWERSHELL 常見問題 (繁體中文)](http://PowerShell-guru.com/faq-PowerShell-in-chinese-traditional/)
7. [如何在遠程伺服器上運行PowerShell命令？](https://read01.com/DQ3M4.html)
8. [PowerShell - Testing if a String is NULL or EMPTY](http://www.morgantechspace.com/2015/04/PowerShell-testing-if-string-is-null-or-empty.html)
9. [執行遠端命令](https://msdn.microsoft.com/zh-tw/PowerShell/scripting/core-PowerShell/running-remote-commands)
10. [Using the Get-Service Cmdlet](https://technet.microsoft.com/en-us/library/ee176858.aspx)
11. [How to pass a param to script block when using invoke-command](https://social.technet.microsoft.com/Forums/windows/en-US/17960e2b-bd47-44fd-b25e-c5092940bf40/how-to-pass-a-param-to-script-block-when-using-invokecommand?forum=winserverPowerShell)
12. [基本操作手冊參考](https://msdn.microsoft.com/zh-tw/PowerShell/scripting/getting-started/basic-cookbooks)
13. [Windows PowerShell 5.0](https://msdn.microsoft.com/en-us/PowerShell/reference/5.0/readme)