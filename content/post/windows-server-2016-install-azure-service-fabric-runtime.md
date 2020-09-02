---
title: "Windows Server 2016 無法安裝 Azure Service Fabric - Runtime 及 SDK"
date: 2017-01-26T00:42:34+08:00
lastmod: 2018-09-10T00:42:34+08:00
draft: false
tags: ["Windows Server 2016","Azure"]
slug: "windows-server-2016-install-azure-service-fabric-runtime"
aliases:
    - /2017/01/windows-server-2016-install-azure-service-fabric-runtime.html
---
# Windows Server 2016 無法安裝 Azure Service Fabric - Runtime 及 SDK
最近看了不少 Azure 服務的介紹以及應用，其中有一些內容提到 Azure Service Fabric，所以想要實際使用並測試看看，結果發現 Windows Server 2016 未被列在官方的支援的作業系統版本中(可以參考[準備您的開發環境](https://docs.microsoft.com/zh-tw/azure/service-fabric/service-fabric-get-started?WT.mc_id=DOP-MVP-5002594))，直覺不可能，所以動手試裝看看

## 安裝 `Service Fabric` Runtime、SDK 和工具

> 僅安裝 Service Fabric Runtime, SDK (不安裝 Visual Studio 工具)
    
按 [下載](http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK) 可以直接下載

## 下載檔會啟動 Web Platform Installer 進行安裝
![install](https://cloud.githubusercontent.com/assets/3851540/22259437/4bae8e8e-e2a0-11e6-96ff-c963f4047782.png)

## 安裝失敗

![error](https://cloud.githubusercontent.com/assets/3851540/22259438/4baed68c-e2a0-11e6-95c2-fe47596b7b76.png)

## 檢視 log

1. log 內容
    
    ``` 
    Product: Windows Azure Service Fabric -- This product requires Microsoft VC Runtime 11 to work.
    ```
2. log 截圖
    
    ![vc11](https://cloud.githubusercontent.com/assets/3851540/22259439/4bb12ca2-e2a0-11e6-8996-2c8cf6b9a016.png)

3. Log 明白地說需要`Microsoft VC Runtime 11`

## 安裝`Microsoft VC Runtime 11`

>嘗試使用新版本 VC 13(2015) 及 VS 14(2017) <span style="color:red">皆無法正常使用，VS 11(2012) Only</span>

- [下載Visual C++ Redistributable for Visual Studio 2012 Update 4](https://www.microsoft.com/en-us/download/confirmation.aspx?id=30679)



# 參考資料
1. [準備您的開發環境](https://docs.microsoft.com/zh-tw/azure/service-fabric/service-fabric-get-started?WT.mc_id=DOP-MVP-5002594)
2. [How To : Install Windows Azure Pack : Service Bus 1.1 (Windows Service Bus 1.1)](http://www.enterpriseframework.com/post/how-to-install-windows-azure-pack-service-bus-1-1-windows-service-bus-1-1)
3. [Visual C++ Redistributable for Visual Studio 2012 Update 4](https://www.microsoft.com/en-us/download/confirmation.aspx?id=30679)