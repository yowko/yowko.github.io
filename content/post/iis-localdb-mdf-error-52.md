---
title: "IIS 存取 LocalDB (.MDF) 時出現 Error 52"
date: 2018-02-04T00:01:00+08:00
lastmod: 2021-11-02T00:36:58+08:00
draft: false
tags: ["IIS","LocalDB","SQL Server"]
slug: "iis-localdb-mdf-error-52"
aliases:
    - /2018/02/iis-localdb-mdf-error-52.html
    - /2018/02/iis-localdb-mdf.html
---
## IIS 存取 LocalDB (.MDF) 時出現 Error 52

資料庫是網頁應用程式儲存資料最常見的方式，存取資料的操作方法也最廣為人知，只是資料庫本身有維護及管理成本，如果目的只是本機開發使用，透過 IIS Express 搭配 LocalDB 就可以滿足大部份情境了，不過一旦需要實際部署至 IIS 上，LocalDB 就不適用了，因為 LocalDB 是 SQL Server Express 的一種特殊執行模式，而 SQL Server Express 本來就不適用在 production 上，加上 LocalDB 也不是設計用來搭配 IIS

剛好最近有個功能想要模擬不同時區的顯示，可是又不想大費周章管理架設 SQL Server，於是就想到透過 LocalDB 來簡單驗證功能正確性，有段時間沒有這麼做了，設定方式有些生疏，順手紀錄一下以備日後又想偷懶XD

## 前提說明

1. 透過 EntityFramework 連線 LocalDB 取得資料做顯示
    * web.config 連線

        ```xml
        <add name="TestDateTimeEntities" connectionString="metadata=res://*/Models.TestDateTime.csdl|res://*/Models.TestDateTime.ssdl|res://*/Models.TestDateTime.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=(LocalDB)\MSSQLLocalDB;attachdbfilename=|DataDirectory|\Database1.mdf;integrated security=True;connect timeout=30;MultipleActiveResultSets=True;App=EntityFramework&quot;" providerName="System.Data.EntityClient" />
        ```

2. local 正常開啟

    ![1localok](https://user-images.githubusercontent.com/3851540/35768879-90ee4880-093d-11e8-92ea-ec235080ad32.png)

3. 部署至遠端 IIS 後出現錯誤

    ![2iiserror](https://user-images.githubusercontent.com/3851540/35768880-9116899e-093d-11e8-81ee-5601f66635e7.png)

## 錯誤訊息

1. 訊息內容
    * 英文

        ```log
        Server Error in '/' Application.
        系統找不到指定的檔案。
        Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 
        
        Exception Details: System.ComponentModel.Win32Exception: 系統找不到指定的檔案。
        
        Source Error: 
        
        An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.
        
        Stack Trace: 
            
        [Win32Exception (0x80004005): 系統找不到指定的檔案。]
        
        [SqlException (0x80131904): A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: SQL Network Interfaces, error: 52 - Unable to locate a Local Database Runtime installation. Verify that SQL Server Express is properly installed and that the Local Database Runtime feature is enabled.)]
        ```

    * 中文

        ```log
        '/' 應用程式中發生伺服器錯誤。
        系統找不到指定的檔案。
        描述: 在執行目前 Web 要求的過程中發生未處理的例外狀況。請檢閱堆疊追蹤以取得錯誤的詳細資訊，以及在程式碼中產生的位置。 
        
        例外狀況詳細資訊: System.ComponentModel.Win32Exception: 系統找不到指定的檔案。
        
        原始程式錯誤: 
        
        在執行目前 Web 要求期間，產生未處理的例外狀況。如需有關例外狀況來源與位置的資訊，可以使用下列的例外狀況堆疊追蹤取得。
        
        堆疊追蹤: 
            
        [Win32Exception (0x80004005): 系統找不到指定的檔案。]
        
        [SqlException (0x80131904): 建立連接至 SQL Server 時，發生網路相關或執行個體特定的錯誤。找不到或無法存取伺服器。確認執行個名稱是否正確，以及 SQL Server 是否設定為允許遠端連線。 (provider: SQL Network Interfaces, error: 52 - 找不到 Local Database Runtime 安裝。請確認 SQL Server Express 已正確安裝且 Local Database Runtime 功能已啟用。)]
        ```

2. 錯誤截圖
    * 英文

        ![3engerror](https://user-images.githubusercontent.com/3851540/35768881-913e335e-093d-11e8-8d4c-8376986d37dd.png)

    * 中文

        ![4zherror](https://user-images.githubusercontent.com/3851540/35768882-9165c68a-093d-11e8-8efe-ca4548cab415.png)

## 解決方式

1. 確認使用 LocalDB 的版本

    * double click --> 開啟 `.mdf`
    * 執行 `SELECT @@VERSION`

        ![5version](https://user-images.githubusercontent.com/3851540/35768883-918d678a-093d-11e8-8d7f-e9301239c6e5.png)

        ```log
        Microsoft SQL Server 2014 - 12.0.2269.0 (X64)   Jun 10 2015 03:35:45   Copyright (c) Microsoft Corporation  Express Edition (64-bit) on Windows NT 6.3 <X64> (Build 10586: )
        ```

2. 下載並安裝 MDF 對應版本 LocalDB
    * SQL Server 2014 - [Microsoft® SQL Server® 2014 Express](https://www.microsoft.com/zh-tw/download/details.aspx?id=42299)

        > 相關介紹：[SQL Server 2014 Express LocalDB](https://msdn.microsoft.com/en-US/library/hh510202%28SQL.120%29.aspx)

    * SQL Server 2016 - [Microsoft® SQL Server® 2016 Service Pack 1 Express](https://www.microsoft.com/zh-tw/download/details.aspx?id=54284)

        > 相關介紹：[SQL Server 2016 Express LocalDB](https://docs.microsoft.com/zh-tw/sql/database-engine/configure-windows/sql-server-2016-express-localdb?WT.mc_id=DOP-MVP-5002594)

## 心得

微軟的檔案下載搞得好難找呀，在 SQL Server 2016 說明頁面上的下載連結是連到 SQL Server 2017，點下載卻只有評估版，要透過搜尋才能找到 SQL Server 2016 Express，使用者體驗有加強的空間

## 參考資訊

1. [Web Application publishing in IIS with .MDF database file](https://forums.asp.net/t/2090050.aspx?Web+Application+publishing+in+IIS+with+MDF+database+file?WT.mc_id=DOP-MVP-5002594)
2. [[LocalDB] Connecting to LocalDB failed [Error Code 52]](http://jaryl-lan.blogspot.tw/2014/08/localdb-connection-to-localdb-failed.html)
3. [SQL Server 2014 Express LocalDB](https://msdn.microsoft.com/en-US/library/hh510202%28SQL.120%29.aspx?WT.mc_id=DOP-MVP-5002594)
4. [SQL Server 2016 Express LocalDB](https://docs.microsoft.com/zh-tw/sql/database-engine/configure-windows/sql-server-2016-express-localdb?WT.mc_id=DOP-MVP-5002594)
