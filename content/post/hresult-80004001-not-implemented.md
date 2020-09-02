---
title: "解決無法使用 IIS Manager 及 AppCmd.exe 列出 Worker Processes 的 Request"
date: 2018-06-25T01:25:00+08:00
lastmod: 2018-10-07T01:25:36+08:00
draft: false
tags: ["Debug","IIS","Windows 10","Windows Server 2016"]
slug: "hresult-80004001-not-implemented"
aliases:
    - /2018/06/hresult-80004001-not-implemented.html
---
# 解決無法使用 IIS Manager 及 AppCmd.exe 列出 Worker Processes 的 Request
最近監控 IIS 上的 application 時發現部份 server 無法列出 request queue，以我的經驗一般情況下並不會特別去看 request queue 的內容，一旦需要看就一定有 server 或是 application 警示出現，如果看不到內容對於除錯就少了可以快速定位發生原因的手段，也就增加偵錯的時間

原本以為是 IIS Manager GUI 的問題，改用 AppCmd.exe 來看內容也出現錯誤，就來看看如何解決吧


## 關於 AppCmd.exe
從 IIS 7 開始加入的命令列工具，可以用來管理 IIS 的重要功能
1. 建立及設定 sites, apps, application pools, 跟 virtual directories
2. 啟動及停止 sites 與回收 application pools
3. 列出執行中的 worker processes 與檢查執行中的 request
4. 搜尋、管理、匯出及匯入 IIS 與 ASP.NET 的設定

* 路徑
    
    > 未加入環境變數，需切換至所在路徑或是使用完整路徑 
    
    ```
    %systemroot%\system32\inetsrv\
    ```

* 基本語法
    
    ```
    APPCMD (command) (object-type) <identifier> < /parameter1:value1 ... >*
    ``` 
    
    > 語法詳細介紹請參考 [Getting Started with AppCmd.exe](https://docs.microsoft.com/zh-tw/iis/get-started/getting-started-with-iis/getting-started-with-appcmdexe?WT.mc_id=DOP-MVP-5002594)

## 問題描述
- 使用 IIS Manager
    1. 開啟 IIS Manager --> 選擇目標 Server --> Worker Processes
        
        ![1openiis](https://user-images.githubusercontent.com/3851540/41821579-90ce455c-7815-11e8-83aa-a0b34e9b35c7.png)
    2. 點擊目標 Application Pool Name 無法顯示 Requests queue (完全沒有反應，也沒有錯誤提示)
        
        ![2workerprocesses](https://user-images.githubusercontent.com/3851540/41821580-90f76c34-7815-11e8-883b-2bdf1c890b21.png)
- 使用 AppCmd.exe
    * 執行 `appcmd.exe list requests TestTimeout` 
    * 錯誤訊息
        * 訊息內容
            
            ```
            ERROR ( hresult:80004001, message:Command execution failed.
            Not implemented
            )
            ``` 
        * 錯誤截圖
            
            ![3cmderror](https://user-images.githubusercontent.com/3851540/41821582-9148a2ac-7815-11e8-9055-93b307e10a29.png)

## 解決方式
- Windows 10
    1. 開啟 `Turn Windows features on or off`
        
        ![4feature](https://user-images.githubusercontent.com/3851540/41821583-9173cae0-7815-11e8-882c-b5168302a5ec.png)
    2. Internet Information Services --> World Wide Web Services --> Health and Diagonostics --> `Request Monitor` , `Tracing`
        
        ![5monitoringandtracing](https://user-images.githubusercontent.com/3851540/41821584-919c3a8e-7815-11e8-85da-69c8f743f079.png) 
- Windows Server
    1. Server Manage --> Manage --> AddRoles amd Features
        
        ![6rolefeature](https://user-images.githubusercontent.com/3851540/41821585-91c7a020-7815-11e8-933b-afac83e6dc1c.png) 
    2. Installation Type --> Role-based or feature-based installation
        
        ![7rolebased](https://user-images.githubusercontent.com/3851540/41821575-901e0da4-7815-11e8-9be4-7aefac72bb0f.png) 
    3. Service Roles --> Web Server(IIS) --> Health and Diagnostics --> `Request Monitor` , `Tracing`
        
        ![8monitortracing](https://user-images.githubusercontent.com/3851540/41821576-9047b8e8-7815-11e8-9bc5-93e174b6ef29.png)
 * 效果
    - IIS Manage GUI
        
        ![9result](https://user-images.githubusercontent.com/3851540/41821577-907943b8-7815-11e8-918f-90394a9d4501.png)
    - AppCmd.exe
        
        ```
        appcmd.exe list request /elapsed:3000
        ``` 
        
        ![10resultappcmd](https://user-images.githubusercontent.com/3851540/41821578-90a3e014-7815-11e8-9cb8-baa4c6c281eb.png)

## 心得
經過設定 IIS 的 Health and Diagnostics 後，已經確認解決問題了，也才還 IIS Manage GUI 的清白 XD
不過一開始發現 IIS Manage GUI 無法查詢又沒有任何訊息，純粹只是無法開啟 request queue 時我真心覺得是 IIS Manage GUI 的問題，直到 AppCmd.exe 拋出錯誤時我才覺得可能是資料來源出問題

雖然 google 一下就可以發現 [Error hresult :80004001 , message : Command execution failed. Not implemented](https://stackoverflow.com/questions/38593856/error-hresult-80004001-message-command-execution-failed-not-implemented) 有正確解決方法，但 IIS Manage GUI 沒有出現任何訊息與 AppCmd.exe 模糊的錯誤訊息，對於解決問題幫助不大呀

# 參考資訊
1. [Getting Started with AppCmd.exe](https://docs.microsoft.com/zh-tw/iis/get-started/getting-started-with-iis/getting-started-with-appcmdexe?WT.mc_id=DOP-MVP-5002594)
2. [Error hresult :80004001 , message : Command execution failed. Not implemented](https://stackoverflow.com/questions/38593856/error-hresult-80004001-message-command-execution-failed-not-implemented)