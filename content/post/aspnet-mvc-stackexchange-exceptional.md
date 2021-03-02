---
title: "如何在 ASP.NET MVC 專案中使用 StackExchange.Exceptional"
date: 2016-12-01T00:42:34+08:00
lastmod: 2021-03-02T00:42:34+08:00
draft: false
tags: ["套件","ASP.NET MVC"]
slug: "aspnet-mvc-stackexchange-exceptional"
aliases:
    - /2016/12/aspnet-mvc-stackexchange-exceptional.html
---
## 如何在 ASP.NET MVC 專案中使用 StackExchange.Exceptional

一套由 `Stack Exchange` 與 `Stack Overflow` 開發於內部使用的 exception 處理套件,可用在 web 與 console 專案上
受 `ELMAH` 啟發，但因 `ELMAH` 不適用於 `Stack Exchange` 與 `Stack Overflow` 在發生網路相關事件時會出現極大量 logging 的特殊需求而自製

## 安裝及設定

1. `Install-Package StackExchange.Exceptional`
2. 修改 web.config  
    - 2-1. `configuration`-->`configSections` 加入 section 定義
  
        ```xml
        <configSections>
            <section name="Exceptional" type="StackExchange.Exceptional.Settings, StackExchange.Exceptional"/>
        </configSections>
        ```

    - 2-2. `configuration` 增加 `Exceptional` 內容

        ```xml
        <Exceptional applicationName="Core">   
            <IgnoreErrors>
                <!-- 勿略的錯誤訊息 (非必要) -->
                <Regexes>
                    <add name="connection suuuuuuuucks" pattern="Request timed out\.$" />
                </Regexes>
                <!-- 勿略的錯誤類型 (非必要), e.g. <add type="System.Exception" /> or -->
                <Types>
                <add type="MyNameSpace.MyException" />
                </Types>
            </IgnoreErrors>
            <!-- log 儲存方式 -->
            <ErrorStore type="Memory" />
            <!-- 其他儲存方式, 常見屬性:
            - rollupSeconds: 非必要 (預設 600 秒), 設定多少時間內的有相同 stack trace 的錯誤會被彙總處理 - 0 表不彙總處理
            - backupQueueSize: 非必要 (預設 1000 則), 設定在所選的儲存方式無法使用時暫存在 Memory 的數量，每兩秒會重試一次 -->
            <!-- JSON: 預設保留數量為 200 則, 會自動刪除超過指定數量的較舊檔案 -->
            <!--<ErrorStore type="JSON" path="~\Errors" size="200" />-->
            <!-- SQL: 直接寫 connectionString 或是 config 中的 connectionStringName 都可以,可以把不同程式的 log 都儲存在同一個 table，但需要有不同的 applicationName (不然無法分辨 log 是哪個 application 的). -->
            <!--<ErrorStore type="SQL" connectionString="Server=.;Database=Exceptions;Uid=Exceptions;Pwd=myPassword!" />-->
            <!--<ErrorStore type="SQL" connectionStringName="MyConnectionString" />-->
            <!-- 支援 MySQL -->
            <!--<ErrorStore type="MySQL" connectionString="Server=.;Database=Exceptions;Username=Exceptions;Pwd=myPassword!" />-->
            <!--<ErrorStore type="MySQL" connectionStringName="MyConnectionString" />-->
        </Exceptional>
        ```

    - 2-3. `configuration` --> `system.webServer` --> `modules` 加入引用  

        ```xml
        <modules>
            <add name="ErrorStore" type="StackExchange.Exceptional.ExceptionalModule, StackExchange.Exceptional" />
        </modules>
        ```

    - 2-4. 畫面顯示設定(非必要)  
    `configuration` --> `system.webServer` --> `handlers` 加入新 handlers

        ```xml
        <handlers>
            <add name="Exceptional" path="exceptions.axd" verb="POST,GET,HEAD" type="StackExchange.Exceptional.HandlerFactory, StackExchange.Exceptional" preCondition="integratedMode" />
        </handlers>
        ```

## 儲存方式

![nologs](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580c9826d814732a5fa2722a/ecdf2a5e8174749fa59f6a2326f11a16/noError_%E7%BB%93%E6%9E%9C.png)

1. 儲存至 `Memory`

    > IIS 重啟，log 會消失

    - 設定方式：

        ```xml
        <ErrorStore type="Memory" />
        ```

    - 實際效果

        >![memory](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580c9826d814732a5fa2722a/2d3bb6552b99e421fdd6be698711c00f/memory_%E7%BB%93%E6%9E%9C.png)

2. json

    - 會耗用磁碟空間(預設200個檔案，超過會自動刪除舊檔案)
    - 記得新增儲存用的資料夾
    - 若路徑不存在會改用 `Memory` 模式
    - 設定方式：

        ```xml
        <ErrorStore type="JSON" path="~\Errors" size="200" rollupSeconds="300" />
        ```

    - 實際效果

        >![jsonfail](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580c9826d814732a5fa2722a/6bdc04a773e92f4940504820926a8d52/jsonfail_%E7%BB%93%E6%9E%9C.png)

3. database
    - 支援 ms-sql 與 my-sql
    - 若無法連線至 database 會改用 `Memory` 模式
    - 使用方式：

        ```xml
        <ErrorStore type="SQL" connectionString="Data Source=.;Integrated Security=False;User ID=username;Password=password;Connect Timeout=15;Encrypt=False;TrustServerCertificate=True;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" />
        ```

    - 實際效果

        >![dbfail](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/580c9826d814732a5fa2722a/16350c0633d3f87c631702151ce40d3e/dbfail1_%E7%BB%93%E6%9E%9C.png)

## 注意事項

`httpRuntime targetFramework`**尚不支援 .NET Framework 4.6.2**

2016/12/01 測試結果可支援至 **.NET Framework 4.6.1**

## 心得

- 優點：
  1. web 及 console 皆適用
  2. 彙總的功能可以迅速發現同類型問題
  3. 如果儲存目標有異常會自動改用 `Memory` (曾經遇過 elmah 儲存至 db，但 db 異常，造成 error 都沒 log 到)
  4. 預設自動刪除舊檔，避免檔案數量過度膨脹 (曾經遇過 elmah 使用 xml 儲存，但上線後一直沒去清檔案，結果資料夾內有數十萬個 xml，檔案總管打開就當了XD)

- 缺點
  1. 文件說明較少
  2. 周邊套件還沒有 ELMAH 充足
  3. 使用 nuget 安裝時，web.config 需要自行加入

## 參考資料

1. [GitHub](https://github.com/NickCraver/StackExchange.Exceptional)
