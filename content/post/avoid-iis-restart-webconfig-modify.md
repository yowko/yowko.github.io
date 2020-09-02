---
title: "修改 web.config 可以不讓 IIS 自動重啟？！"
date: 2018-11-14T00:42:34+08:00
lastmod: 2018-11-14T00:42:34+08:00
draft: false
tags: ["IIS","ASP.NET MVC","ASP.NET WEB API","web.config"]
slug: "avoid-iis-restart-webconfig-modify"
---
# 修改 web.config 可以不讓 IIS 自動重啟？！
這是同事提出的問題：想要修改 web.config 中的 NLog log level，但不敢冒然修改 production server 上的檔案，深怕檔案一改立馬引發 IIS recycle 直接影響線上實際使用者。

雖然依著過去經驗：只要修改 web.config 就會觸發 IIS restart，但為了避免我自己學藝不精而誤導同事還是查查資料確認一下，結果看到有網友提出一些方法可以讓 IIS 不會因為 web.config 的修改而重啟，立馬來看看是不是真的這麼神奇？！

## 環境設定
1. Windows 10 - 1803 (OS Build 17134.345)

    > 使用 IIS 10
2. 透過之前筆記 [無法參考 App_Code 資料夾下的 class](https://blog.yowko.com/access-class-in-app-code/) 提到的技巧在 website 啟動時紀錄時間

## 原始情境
1. Home/Index 會顯示出網站啟動時間與頁面載入時間

    > 按下 f5 後：網站啟動時間不會改變，頁面載入時間則持續變動

    ![1starttime](https://user-images.githubusercontent.com/3851540/48495581-487b8780-e86b-11e8-8a15-69992fb58cbe.gif)

2. 隨意在 web.config 中加上空白字元後儲存

    > 修改 web.config 後，即可發現網站啟動時間已改變

    - 修改前
        
        ![2beforeconfig](https://user-images.githubusercontent.com/3851540/48494825-98f1e580-e869-11e8-8eaf-99d3cc764f27.png) 
    - 修改後    

        ![3afterconfig](https://user-images.githubusercontent.com/3851540/48494826-98f1e580-e869-11e8-880a-a21beb170c95.png)

## 避免 IIS 重啟的可能做法

- 修改 web.config

    > 設定可以針對單一站台

    - `fcnMode`

        > fcnMode 是 .NET Framework 4.5 開始加入的屬性，詳細資訊可以參考 [關於 ASP.NET File Change Notification (FCN) 的一些事](https://blog.yowko.com/fcn/)
            
        ```xml
        <system.web>
            <httpRuntime fcnMode="Disabled"/>
        </system.web>
        ``` 
    - `waitChangeNotification` 與 `maxWaitChangeNotification`
        - waitChangeNotification

            - 單位：`秒`
            - 預設值：`0`
            - 作用：設定變更通知觸發 AppDomain 重啟的等待時間
        - maxWaitChangeNotification
            - 單位：`秒`
            - 作用：設定變更通知與 AppDomain 重啟的最大時間間隔
            - 建議值：應大於部署時複製檔案的時間
        
        ```xml
        <system.web>
            <httpRuntime waitChangeNotification="315360000" maxWaitChangeNotification="315360000"/>
        </system.web>
        ```
 
- 修改系統設定

    >-  設定針對機器，會套用至所有 IIS 站台
    >- 新增 `\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\ASP.NET\FCNMode` 值為 `1`

    - 透過 `regedit`

        ![4regedit](https://user-images.githubusercontent.com/3851540/48494827-998a7c00-e869-11e8-972e-ca6fc72be3a9.png)

    - 使用 `PowerShell` 

        ```ps1
        New-ItemProperty -Path "hklm:\SOFTWARE\WOW6432Node\Microsoft\ASP.NET" -Name "FCNMode" -Value 1 -PropertyType DWORD
        ``` 

    * 完成設定可以透過 `iisreset /restart` 或是 `重新開機` 來套用設定
- 修改 IIS - ApplicationPool 設定

    1. IIS Manager --> Application Pools --> 目標 application 右鍵 --> Advanced Settings

        ![5iissettings](https://user-images.githubusercontent.com/3851540/48494828-998a7c00-e869-11e8-8bf1-a57815358e41.png)
    
    2. Disable Recycling for Configuration 改為 `True`
    
        ![6disablerecycle](https://user-images.githubusercontent.com/3851540/48494829-998a7c00-e869-11e8-88b8-b604e902ea17.png)


## 實際結果
     
- 修改 web.config

    - `fcnMode`

        > 除了 `web.config` 外的變更皆會忽略 (e.g. bin 資料夾中的檔案異動、global.asax 的修改...)，但修改 `web.config` 依然會立即觸發 recycle
    
    - `waitChangeNotification` 與 `maxWaitChangeNotification`

        > 大致行為與修改 `fcnMode` 相同，但需要特別留意設定 `waitChangeNotification` 與 `maxWaitChangeNotification` 後再修改 web.config 會出現站台 hang 住完全無法回應的情況 (我自己是執行了兩次 `iisreset /restart` 才讓站台重新恢復正常 - 第一次會出現 stop failed)

        ![7pending](https://user-images.githubusercontent.com/3851540/48494830-9a231280-e869-11e8-8029-4abd39bc073e.png)

- 修改系統設定

    > 行為與修改 web.config 中的 `fcnMode` 相同

- 修改 IIS - ApplicationPool 設定
    
    > 這是個針對 ApplicationPool 設定變更時的處置動作屬性，預設為 'false' - 表示允許在 ApplicationPool 設定變更時進行 recycle，因此 web.config 的變更並不歸這個設定管理，只有 ApplicationPool 相關設定 (e.g. 是否啟用 32-bit...之類) 才會受這個屬性值影響

## 心得
在驗證是否可以有方法達成 web.config 修改不引發 IIS 重啟的做法時心情轉折很大，依過去經驗沒有根據地認定 web.config 的修改一定會造成站台 recycle，但查詢之後又有人提到幾種做法可以避免，實際測試後反而又證明了過去經驗沒錯，雖然也想過在舊的 IIS 版本可能真的可行，但最後放棄實際找舊 IIS 來驗證，畢竟現在的版本才是重點，過去再怎麼美好都已經過去了 哈哈

以個人測試結果來說：<span style="color:red">在 IIS 10 上沒有辦法有效避免修改 web.config 而造成的 IIS recycle</span>

# 參考資訊
1. [How to prevent an ASP.NET application restarting when the web.config is modified?](https://stackoverflow.com/questions/613824/how-to-prevent-an-asp-net-application-restarting-when-the-web-config-is-modified)
2. [關於 ASP.NET File Change Notification (FCN) 的一些事](https://blog.yowko.com/fcn/)
3. [HttpRuntimeSection.WaitChangeNotification Property](https://docs.microsoft.com/en-us/dotnet/api/system.web.configuration.httpruntimesection.waitchangenotification?WT.mc_id=DOP-MVP-5002594)
4. [HttpRuntimeSection.MaxWaitChangeNotification Property](https://docs.microsoft.com/en-us/dotnet/api/system.web.configuration.httpruntimesection.maxwaitchangenotification?WT.mc_id=DOP-MVP-5002594)
5. [IIS Application Domain and Pool Recycling](https://www.treeloop.com/blog/iis-application-domain-and-pool-recycling)
6. [ApplicationPoolRecycling.DisallowRotationOnConfigChange Property](https://msdn.microsoft.com/en-us/library/microsoft.web.administration.applicationpoolrecycling.disallowrotationonconfigchange%28v=vs.90%29.aspx?f=255&MSPPError=-2147217396)