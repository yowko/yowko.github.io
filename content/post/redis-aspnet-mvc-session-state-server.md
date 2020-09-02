---
title: "使用 Redis 當做 ASP.NET MVC 的 Session State Server"
date: 2017-01-27T00:42:34+08:00
lastmod: 2020-09-01T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC","Redis"]
slug: "redis-aspnet-mvc-session-state-server"
aliases:
    - /2017/01/redis-aspnet-mvc-session-state-server.html
---
# 使用 Redis 當做 ASP.NET MVC 的 Session State Server
Redis 是一套 Open Source ，Memory base 的 Key-Value 儲存資料庫，由於效能優異而受到重視。今天來看看如何使用 Redis 來當做 ASP.NET MVC 的 Session State Server.

## 安裝 Redis
1. 下載 Redis on Winodws
    
    > Redis 官方沒有支援 Windows，建議使用 linux 環境來架設 (Windows 10 user 可以使用 Linux subsystem 來模擬)，今天就使用 MSOpenTech porting 的版本來簡單紀錄 ( Redis on Winodws 一直沒有持續更新，正式 release 停留在 3.0.504 版要特別留意)

    [Release of Redis on Windows from Github](https://github.com/MSOpenTech/redis/releases)

2. 直接安裝
    - 預設使用 `6379` port
    - 加入防火牆例外清單
        
        ![1DEFAULT](https://cloud.githubusercontent.com/assets/3851540/22259834/d039e5d0-e2a1-11e6-857f-a81d9870816c.png)

3. 設定使用記憶體上限
    - 依實際使用況狀來設定 Redis 的記憶體使用上限
    
        ![2MEMORY](https://cloud.githubusercontent.com/assets/3851540/22259836/d0597b98-e2a1-11e6-8797-88b923d7b7a9.png)

## 專案安裝套件
以下安裝方式擇一即可

1. NuGet GUI
	- 專案右鍵 --> Manage NuGet Packages...
	    
        ![3rightclick](https://cloud.githubusercontent.com/assets/3851540/22259835/d04c60a2-e2a1-11e6-891a-6fb625e2d880.png)

	- 搜尋 `Microsoft.Web.RedisSessionStateProvider`
        
        ![4search](https://cloud.githubusercontent.com/assets/3851540/22259837/d05edc6e-e2a1-11e6-84a6-b2fa185703c6.png)
    
2. Package Manager Console
	- Visual Studio 主選單 Tools --> NuGet Package Manager --> Package Manager Console
	    
        ![5console](https://cloud.githubusercontent.com/assets/3851540/22259829/d01f9478-e2a1-11e6-8e5e-d091547e6e59.png)

	- 安裝 `Install-Package Microsoft.Web.RedisSessionStateProvider`
	    
        ![6consoleinstall](https://cloud.githubusercontent.com/assets/3851540/22259831/d024c6e6-e2a1-11e6-9adc-61e64f80318d.png)

## 設定連線資訊
- 安裝完成後會直接在專案上加入參考，也會在 Web.config 加上設定區塊跟範例
        
     ```xml
     <system.web> 
     <sessionState mode="Custom" customProvider="MySessionStateStore">
         <providers>
             <!-- For more details check https://github.com/Azure/aspnet-redis-providers/wiki -->
             <!-- Either use 'connectionString' OR 'settingsClassName' and 'settingsMethodName' OR use 'host','port','accessKey','ssl','connectionTimeoutInMilliseconds' and 'operationTimeoutInMilliseconds'. -->
             <!-- 'throwOnError','retryTimeoutInMilliseconds','databaseId' and 'applicationName' can be used with both options. -->
             <!--
             <add name="MySessionStateStore" 
                 host = "127.0.0.1" [String]
                 port = "" [number]
                 accessKey = "" [String]
                 ssl = "false" [true|false]
                 throwOnError = "true" [true|false]
                 retryTimeoutInMilliseconds = "5000" [number]
                 databaseId = "0" [number]
                 applicationName = "" [String]
                 connectionTimeoutInMilliseconds = "5000" [number]
                 operationTimeoutInMilliseconds = "1000" [number]
                 connectionString = "<Valid StackExchange.Redis connection string>" [String]
                 settingsClassName = "<Assembly qualified class name that contains settings method specified below. Which basically return 'connectionString' value>" [String]
                 settingsMethodName = "<Settings method should be defined in settingsClass. It should be public, static, does not take any parameters and should have a return type of 'String', which is basically 'connectionString' value.>" [String]
                 loggingClassName = "<Assembly qualified class name that contains logging method specified below>" [String]
                 loggingMethodName = "<Logging method should be defined in loggingClass. It should be public, static, does not take any parameters and should have a return type of System.IO.TextWriter.>" [String]
                 redisSerializerType = "<Assembly qualified class name that implements Microsoft.Web.Redis.ISerializer>" [String]
             />
             -->
             <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="" accessKey="" ssl="false" />
         </providers>
         </sessionState>
     </system.web>
     ```
    
- 出現 No connection is available 錯誤
    - 錯誤訊息

        ```
        No connection is available to service this operation: EVAL
        ```
    - 錯誤截圖
        ![7sslerror](https://cloud.githubusercontent.com/assets/3851540/22259830/d02415b6-e2a1-11e6-9b1e-262dba2f2907.png)
        
    > nuget 安裝後會直接在 web.config 加上連線設定 `MySessionStateStore`，<span style="color:red">預設啟用 ssl</span>, 如果 redis server 沒有設定啟用 ssl 就會出現下列錯誤，請記得將 ssl 改為 false (上面範例我已調整不使用 ssl)


- Redis 連線設定
 
    屬性 |預設值|說明
    ---|:---|:---
    host| -| Redis Server 的 IP 或是 Server Name
    port| -| Redis Server 服務的 port
    accessKey|-|Redis 啟用授權時 的 Redis Server 密碼
    ssl|false|啟用 SSL 連線至 Redis Server 與否
    throwOnError|true|事件失敗時擲出例外
    retryTimeoutInMilliseconds|0|失敗時以指定的毫秒數重試，0表不重試
    databaseId|0|指定 database
    applicationName|-|key 以 `{<Application Name>_<Session ID>}_Data` 格式儲存，可讓不同 application 用相同的 key
    connectionTimeoutInMilliseconds|5000|連線逾時毫秒數
    operationTimeoutInMilliseconds|1000|同步逾時毫秒數

## 確認 Redis 資料
1. 先加一個 session 資料
    
    ![9addsession](https://cloud.githubusercontent.com/assets/3851540/22259833/d02bbc76-e2a1-11e6-88de-3b7b163db1a4.png)

2. 使用  redis-cli.exe 連線 Redis
    - 預設路徑 `C:\Program Files\Redis\redis-cli.exe`
    - 連線語法
        
        ```
        "C:\Program Files\Redis\redis-cli.exe" -h 127.0.0.1
        ```

3. 列出所有 key
 	
    ```
    KEYS *
    ```

4. 檢查特定 key 內容
    
    ```
    HGETALL key
    ```
    ![8result](https://cloud.githubusercontent.com/assets/3851540/22259832/d024ff4e-e2a1-11e6-9207-b1b34a723bdc.png)
 

# 參考資料
1. [Announcing ASP.NET Session State Provider for Redis Preview Release](https://blogs.msdn.microsoft.com/webdev/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release/)
2. [Redis 官網](https://redis.io)
3. [Release of Redis on Windows from Github](https://github.com/MSOpenTech/redis/releases)
4. [Redis On Windows - Part Two](http://www.c-sharpcorner.com/article/redis-on-windows-part-two/)
5. [Azure Redis 快取的 ASP.NET 工作階段狀態提供者](https://docs.microsoft.com/zh-tw/azure/redis-cache/cache-aspnet-session-state-provider?WT.mc_id=DOP-MVP-5002594)