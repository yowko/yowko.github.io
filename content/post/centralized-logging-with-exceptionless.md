---
title: "使用 Exceptionless 取代 ELK(Elasticsearch, Logstash, Kibana) 中的 kibana 來完成 log 集中化"
date: 2017-03-01T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ELK"]
slug: "centralized-logging-with-exceptionless"
aliases:
    - /2017/03/centralized-logging-with-exceptionless.html
---
## 使用 Exceptionless 取代 ELK(Elasticsearch, Logstash, Kibana) 中的 kibana 來完成 log 集中化

目前公司的 application logs 集中化是用 ELK(Elasticsearch, Logstash, Kibana) 來處理的，而前公司則使用付費軟體 Splunk 來做，兩者差別最明顯的就是 ELK 是 open source 軟體 ;Splunk 是商業軟體，功能設定的操作上是比較簡易的; 那為什麼有了 ELK 還需要 Exceptionless 呢？ 主因是 kibana 在權限管理的套件需要額外付費，不包含在 open source 的範圍，加上 Exceptionless 是 .Net 開發的，對於微軟派開發人員是比較容易修改及客製的，因此為了可以在 user interface 這層具備基本權限功能打算使用 Exceptionless 來取代。

在 2016 年時曾經推薦給同事使用，但後來同事未採納：原因是在看了官網 [Exceptionless 官網](https://exceptionless.com/) 介紹後認為是雲端服務，免費使用量只保留三天內的事件、一個月還只能存 3000 筆紀錄;事實上 Exceptionless 是 open source 的軟體，如果不想自己架的話可以考慮他們的雲端服務，而我們則是想用採取 self host 的作法，相關文件可以參考 [Exceptionless Self Hosting](https://github.com/exceptionless/Exceptionless/wiki/Self-Hosting)

2016 年時我也曾經嘗試自行架設過測試環境，雖然有成功架起來，但當時 Exceptionless 3.x 僅支援 Elasticsearch 1.7 版本，由於公司已使用 Elasticsearch 3.x 實際助益不大，直到 2017-01 的 Exceptionless 4 才一下跳躍式支援到 Elasticsearch 5

## 基本需求

[Exceptionless Self Hosting](https://github.com/exceptionless/Exceptionless/wiki/Self-Hosting) 針對不同的環境有不同的要求，以下文章內容將使用測試環境展示

1. 測試環境
    - .NET 4.6.1
    - Java JDK 1.8+
    - IIS Express 8+
    - PowerShell 3+

    > <span style="color:red">注意：測試環境的設定僅限測試用，不應該用在正式環境上</span>

2. 正式環境
    - .NET 4.6.1
    - IIS 7.5+
    - ElasticSearch 5.1

## 如何安裝

1. 從 [github](https://github.com/exceptionless/Exceptionless/releases) 下載 Exceptionless zip
2. 解壓縮 Exceptionless.zip
3. 確定 PowerShell 有執行權限
    - 以 administrator 開啟 command promt
    - 輸入 `powershell Set-ExecutionPolicy Unrestricted` 並執行
4. double click start.bat 進行安裝

    ![1start](https://cloud.githubusercontent.com/assets/3851540/23407878/e8767a1c-fe00-11e6-9b9d-b265ea477998.png)
    - 會使用 powershell 下載並安裝 elasticsearch (預設使用 `9200` port)

        ![2downloadelastic](https://cloud.githubusercontent.com/assets/3851540/23407880/e879b3c6-fe00-11e6-90e4-61db1486b3d6.png)

        ![3startelastic](https://cloud.githubusercontent.com/assets/3851540/23407877/e8747ed8-fe00-11e6-816b-d290100b1332.png)
    - 接著下載並安裝 kibana server (預設使用 `5601` port)

        ![4downloadkibana](https://cloud.githubusercontent.com/assets/3851540/23407879/e8786a84-fe00-11e6-8191-c1aade5eea3b.png)

        ![5startkibana](https://cloud.githubusercontent.com/assets/3851540/23407881/e8842f5e-fe00-11e6-81f5-cf2e15be6353.png)
    - 啟動 iis express (預設使用 `50000` port)

        ![6startwebsite](https://cloud.githubusercontent.com/assets/3851540/23407882/e886928a-fe00-11e6-9b7f-2462511155df.png)

        ![7startwebsite](https://cloud.githubusercontent.com/assets/3851540/23407884/e8aa8d2a-fe00-11e6-8c72-2b43eaea526d.png)

## 相關設定

1. 首次登入需註冊使用者

    ![8singup](https://cloud.githubusercontent.com/assets/3851540/23407883/e8a0df00-fe00-11e6-8c0c-6a91194dfe1d.png)
2. 設定專案資訊

    ![9setupproject](https://cloud.githubusercontent.com/assets/3851540/23407885/e8af3bae-fe00-11e6-9df6-f72fc442b8dc.png)
3. 選擇專案類型

    ![9projectype](https://cloud.githubusercontent.com/assets/3851540/23407873/e812549c-fe00-11e6-95e0-3847aae430d2.png)
4. 會提供如何使用的說明及 api key

    ![10howtouse](https://cloud.githubusercontent.com/assets/3851540/23407874/e8489f84-fe00-11e6-8621-e54089053ffd.png)

## 如何使用 (以 ASP.NET MVC 為例)

1. 安裝 Exceptionless.Mvc 套件(擇一即可)
    - 使用 NuGet Manager GUI
        - 搜尋 `Exceptionless.Mvc.Signed`

             ![11nugetinstall](https://cloud.githubusercontent.com/assets/3851540/23407876/e85f265a-fe00-11e6-8100-e00569e1bb4e.png)
    - 使用 NuGet console

        ```cs
        Install-Package Exceptionless.Mvc 
        ```

2. 在 web.config 加入 apiKey 及 url

    ```xml
    <exceptionless apiKey="API_KEY_HERE" serverUrl="http://localhost:50000"/>
    ```

3. 手動拋出錯誤

    ```cs
    ex.ToExceptionless().Submit()
    ```

4. 其他送出事件的方式可以參考 [Sending Events](https://github.com/exceptionless/Exceptionless.Net/wiki/Sending-Events)

     > 個人偏好的方式是使用 nlog or log4net，就可以不用增加寫 log 的程式碼直接多個 log target 把 log 傳至 elasticsearch 去

    - 以 nlog 為例
        - 安裝 Exceptionless.NLog
        - 在 nlog 的 config 加上 exceptionless extension

            ```xml
            <extensions>
                <add assembly="Exceptionless.NLog"/>  
            </extensions>
            ```

        - 在 nlog 的 config 加上 exceptionless 的 target 設定

            ```xml
            <target name="exceptionless" apiKey="API_KEY_HERE" serverUrl="http://localhost:50000" xsi:type="Exceptionless">
                <field name="host" layout="${machinename}"/>
                <field name="identity" layout="${identity}" />
                <field name="windows-identity" layout="${windows-identity:userName=True:domain=False}" />
                <field name="process" layout="${processname}" />
            </target>
            ```

        - 在 nlog 的 config 加上 exceptionless 的 rules 設定

            ```xml
            <logger name="*" minlevel="Trace" writeTo="exceptionless" />
            ```

        - 程式使用方式

            ```cs
            private Logger logger = LogManager.GetCurrentClassLogger();
            logger.Debug("test123");
            ```

        - 實際效果

            ![12result](https://cloud.githubusercontent.com/assets/3851540/23407875/e85da866-fe00-11e6-9daf-e499e3fb16b8.png)

## 心得

舊版 Exceptionless 設定的工很多又很容易出錯，現在的版本難度降低非常多，可以再深入試試看

## 參考資料

1. [Exceptionless 官網](https://exceptionless.com/)
2. [Exceptionless Self Hosting](https://github.com/exceptionless/Exceptionless/wiki/Self-Hosting)
3. [Sending Events](https://github.com/exceptionless/Exceptionless.Net/wiki/Sending-Events)
