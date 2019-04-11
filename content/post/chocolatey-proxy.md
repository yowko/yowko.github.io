---
title: "為 Chocolatey 設定需驗證的 proxy"
date: 2016-12-31T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "chocolatey-proxy"
aliases:
    - /2016/12/chocolatey-proxy-with-authentication.html
---
# 為 Chocolatey 設定需驗證的 proxy
身為一個 windows 開發人員是不是有時會想把 nuget 功能擴大到 Visual Studio 之外的地方呢？！像 NPM 一樣，只要有 command line 就可以安裝程式，還不限於 Visual Studio 的相關功能？！Chocolatey 就可以達到這個目標了！


## 安裝 Chocolatey
1. 使用 commnad line(Cmd.exe)

    ```
    @powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
    ``` 

2. 使用 commnad line(Cmd.exe) 並指定 proxy

    ```
    @powershell -NoProfile -ExecutionPolicy Bypass -Command "[System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH="%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
    ```    


3. 使用 powershell
下載 [install.ps1](https://chocolatey.org/install.ps1), 執行安裝

## 設定 Chocolatey proxy
1. 使用 Command Line
    
    ```
	choco config set proxy http://proxyserver:proxyport  
	choco config set proxyUser UserName   
	choco config set proxyPassword password
    ```
    
    ![setting](https://trello-attachments.s3.amazonaws.com/581164786236a0546ab45acf/677x442/089d48974a1c2d43dbc515260a592ca2/setting1_%E7%BB%93%E6%9E%9C.png)


2. 使用 PowerShell
- 2-1. 開啟 PowerShell Commnand Line
- 2-2. 依序設定
    
    ```
    $env:chocolateyProxyLocation='http://proxyserver:proxyport'
    $env:chocolateyProxyUser = 'AD\UserName'
    $env:chocolateyProxyPassword = 'password'
    ```
    
    ![setting2](https://trello-attachments.s3.amazonaws.com/581164786236a0546ab45acf/924x132/4b1bf5a5e69af9e11e8fa1db4e3247df/set2_%E7%BB%93%E6%9E%9C.png)
 
## 確認設定
1. 使用 Command Line
    
    ```
	choco config get proxy
	choco config get proxyUser
	choco config get proxyPassword
    ```
    
    ![get1](https://trello-attachments.s3.amazonaws.com/581164786236a0546ab45acf/677x442/8600088b3f20582a178dfd5cbf1857b0/get1_%E7%BB%93%E6%9E%9C.png)

2. 使用 PowerShell
	
    ```
    $env:chocolateyProxyLocation
    $env:chocolateyProxyUser
    $env:chocolateyProxyPassword
    ```
    
    ![get2](https://trello-attachments.s3.amazonaws.com/581164786236a0546ab45acf/695x187/9a2f84368f70304ea037f0910c349913/get2_%E7%BB%93%E6%9E%9C.png)

# 解除設定
1. Command Line

    ``` 
	choco config unset proxy
	choco config unset proxyUser
	choco config unset proxyPassword
    ```
    
    ![unset1](https://trello-attachments.s3.amazonaws.com/581164786236a0546ab45acf/677x442/11c48cfa2edf6debea91b41feb6ab90f/unset1_%E7%BB%93%E6%9E%9C.png)

2. PowerShell

    ```
	$env:chocolateyProxyLocation=''
    $env:chocolateyProxyUser=''
    $env:chocolateyProxyPassword=''
    ```
    
    ![unset2](https://trello-attachments.s3.amazonaws.com/581164786236a0546ab45acf/585x137/df0c159db98a4e7166f13fc77c5d3a34/unset2_%E7%BB%93%E6%9E%9C.png)
 
# 參考資料
1. [安裝](https://chocolatey.org/install)
2. [GitHub Wiki](https://github.com/chocolatey/choco/wiki/Proxy-Settings-for-Chocolatey)
3. [Config Command](https://chocolatey.org/docs/commands-config)
4. [Chocolatey packages](https://chocolatey.org/packages)