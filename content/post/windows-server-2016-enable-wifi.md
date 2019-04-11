---
title: "啟用 Windows Server 2016 的無線網路"
date: 2016-12-28T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["Windows Server 2016"]
slug: "windows-server-2016-enable-wifi"
aliases:
    - /2016/12/windows-server-2016-enable-wifi.html
---
# 啟用 Windows Server 2016 的無線網路
Windows Server 2016 安裝好了，但卻沒有網路，原本以為新的作業系統都會內建網路卡驅動程式的，想不到竟然來了個網路無法用的狀況，只好從別台電腦上網抓完再傳回去安裝，差點忘記還得用 USB 傳XD，電腦不能連網還真是麻煩。      

搞了大半天連主機板晶片的都更新了，網路還是不能用，難道 Windows Server 2016 太新？沒有合適可用的驅動？！還是我人品問題跟 Windows Server 2016 命中註定不合？！ 

實在搞不定，已經放棄了，結果幾個小時過後靈機一動，突然想到會不會是 Server 預設是不開啟無線網路，畢竟 Server 本來就該用超快的有線網路嘛，google 一下果真證實我的想法，立馬留個紀錄。

## 1. 無法啟用無線網路

![icon](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/495x193/2782846599908f57e251d2c40696d35a/output_iconfial.png)

- 裝置管理員
    
    ![device manager](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/581f4a2629cd282a620beebe/5efbd03e481fc8e4dbe6fceb71657f5d/output_devicefail.png)

## 2. 開啟 PowerShell 
- 建議使用 PowerShell ISE ，可以享受 `intellisense`
    
    ![powershell ise](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/491x785/de37cdcd54fcacfc1d54635c71133c95/output_powershell.png)


## 3. 確認無線網路服務的狀態
- `Get-WindowsFeature *Wireless*`
    
    ![intellisense](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/660x275/733d7ba84cc730f33ca7c9a91e011d19/output_intellisence.png)
    
    ![command](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/605x245/03f2be0192225f80b4c6f41d2d06fd58/output_getstatus.png)

- 收集資料
    
    ![gathering](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/737x310/3f5f511bc96cd47c600d12c22207e58c/output_gatheringdata.png)

- 服務未安裝
    
    ![notinstall](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/1012x313/b146effaa410f2c3977e017bc2d027f6/output_statusavailable.png)

## 4. 安裝無線網路服務
- `Install-WindowsFeature -Name Wireless-Networking`
    
    ![intellisense](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/962x538/c8f996f1559c07599daf24a1e10a66de/output_installing.png)
    
    ![command](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/958x333/abca44322ac50b2dc77f238658bc6e0c/output_installcommand.png)

- 安裝中
    
    ![installing](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/738x305/66c01115f08e0f78129f64ee86129cf2/output_installingstatus.png)

-  服務已安裝
    - 確認狀態 `Get-WindowsFeature *Wireless* `
        
        ![command](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/605x245/03f2be0192225f80b4c6f41d2d06fd58/output_getstatus.png)
    
    - 已安裝
        
        ![INSTALLED](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/1130x330/405ebd0bdbc3a0d1767090ab2aa2ba43/output_statusinstalled.png)

## 5. 需要重開機
- 需重啟才能完成重裝程序
    
    ![restart](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/997x443/9a8a7840b8ffbe942eae8241c1df2ef3/output_needrestart.png)


## 6. 啟動服務
- `net start WlanSvc`
    
    ![start](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/953x307/24efecb34ea523582edabf582f055de1/output_install.png)

- 已啟動
    
    ![started](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/968x375/8a2a5c4e51a6d8eb399a59047573b459/output_installed.png)

## 7. 無線網路已安裝

![icon](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/423x130/186a6b43fad3e5cb861ea135e39c7cae/output_iconok.png)

- 裝置管理員
    
    ![deviceok](https://trello-attachments.s3.amazonaws.com/581f4a2629cd282a620beebe/1200x819/331cad6c7a9a4dbd867cb05bd4764072/output_deviceok.png)

# 參考資料
1. [How to Enable Wireless in Windows Server 2016?](http://www.technig.com/enable-wireless-windows-server/)