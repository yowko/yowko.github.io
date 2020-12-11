---
title: "使用 Common.Logging 搭配 NLog 及 log4net"
date: 2017-07-14T23:52:00+08:00
lastmod: 2020-12-11T23:52:50+08:00
draft: false
tags: ["套件","log4net","NLog"]
slug: "common-logging"
aliases:
    - /2017/07/common-logging.html
---
# 使用 Common.Logging 搭配 NLog 及 log4net
在新專案中，同事打算統一 log 的紀錄方式，所以繼承了 log4net 並在 log 的 api 上加入自訂行為，讓後續 log 餵進 ELK 時可以比較順利

出發點當然是為了日後處理方便，但我卻覺得不合理，因為 ELK 只需要 log 紀錄的方式符合特定 pattern 即可，不必為此限定專案只能使用 log4net ，像之前部份專案已經使用 NLog，難道要為了配合餵 log 而整個換 log 系統嗎？

後來想起 Common.Logging 這個套件可以使用一致 log 語法，透過設定將 log 轉由 NLog 或是 log4net 處理，就來看看可以如何使用吧

## 關於 Common.Logging

1.  支援多種 logging framework
    *   log4net (v1.2.9 - v1.2.15)
    *   NLog (v1.0 - v4.1)
    *   Microsoft Enterprise Library Logging Application Block (v3.1 - v6.0)
    *   Microsoft AppInsights
    *   Microsoft Event Tracing for Windows (ETW)*
    *   Log to STDOUT
    *   Log to DEBUG OUT

2.  支援多個 .NET 執行環境
    *   .NET 2.0
    *   .NET 3.0
    *   .NET 3.5
    *   .NET 4.0
    *   .NET 4.5
    *   Silverlight 5.0
    *   Windows Phone 7.x
    *   Windows Phone 8.x
    *   WinRT 8.1 (for Windows 8.1 and Windows Phone 8.1)
    *   Universal Windows Platform 10.0+ (WinRT for Windows 10)
    *   .NET Core 1.0*

        > 我自行測試 .NET 4.6 也 OK

3.  採用 Apache License, Version 2.0 授權
4.  詳細資訊可以參考 [Common-logging](http://net-commons.github.io/common-logging/)
5.  程式碼可以參考 [net-commons/common-logging](https://github.com/net-commons/common-logging)


## 安裝 Common.Logging

1.  安裝 Common.Logging
    *   該步驟可以省略(因下一步驟包含 Common.Logging 相依，預設會安裝)
    *   以 Package Management Console 為例

        > `Install-Package Common.Logging`

2.  安裝 Common.Logging 對應 logging framework 套件


    > 需特別留意套件名稱上的版號，需要與 logging framework 配合才能使用

    *   以 NLog 為例 (2017/07/14 NLog 最新版為 4.1.2)

        > `Install-Package Common.Logging.NLog41`

    *   以 log4net 為例 (2017/07/14 log4net 最新版為 1.2.11)

        > `Install-Package Common.Logging.log4net1211`

## 設定 web.config

1.  加上 config section 定義

    > 如果對 config section 有興趣，可以參考 [使用 ConfigurationSection 自訂 ASP.NET config (web.config) 區段](/2017/02/use-configurationsection-customize-aspnet-config.html)

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <configSections>
            <sectionGroup name="common">
                <section name="logging" type="Common.Logging.ConfigurationSectionHandler, Common.Logging" />
            </sectionGroup>
        </configSections>
    </configuration>
    ```
    *   使用 NLog

        > 需加上 `<section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />`

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <configSections>
                <sectionGroup name="common">
                    <section name="logging" type="Common.Logging.ConfigurationSectionHandler, Common.Logging" />
                </sectionGroup>
                <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
            </configSections>
        </configuration>
        ```
    *   使用 log4net

        > 需加上 `<section name="log4net" type="log4net.Config.log4netConfigurationSectionHandler,log4net" />`

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <configSections>
                <sectionGroup name="common">
                    <section name="logging" type="Common.Logging.ConfigurationSectionHandler, Common.Logging" />
                </sectionGroup>
                <section name="log4net" type="log4net.Config.log4netConfigurationSectionHandler,log4net" />
            </configSections>
        </configuration>
        ```
2.  加上 logging framework 對應設定

    > *   type 設定需指定正確 Common.Logging 對應 logging framework 套件版本
    >     *   configType 有四種值：
    > 
    >     *   INLINE：設定直接寫在 app.config 或是 web.config
    >     *   FILE：使用單獨的 config 檔
    >     *   FILE-WATCH：使用單獨 config 檔並監控變化
    >     *   EXTERNAL：log4net 的設定是由別處處理，標記為略過的意思

    *   以 NLog 為例 (2017/07/14 NLog 最新版為 4.1.2)

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <common>
                <logging>
                <factoryAdapter type="Common.Logging.NLog.NLogLoggerFactoryAdapter, Common.Logging.NLog41">
                    <arg key="configType" value="INLINE" />
                </factoryAdapter>
                </logging>
            </common>
            <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd" autoReload="true" throwExceptions="false" internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
                <targets async="true">
                    <target xsi:type="File" name="f" fileName="${basedir}/logs-inline/${shortdate}.log" layout="${longdate} ${uppercase:${level}} ${message}" />
                </targets>
                <rules>
                    <logger name="*" minlevel="Debug" writeTo="f" />
                </rules>
            </nlog>
        </configuration>
        ```
    *   以 log4net 為例 (2017/07/14 log4net 最新版為 1.2.11)

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <common>
                <logging>
                    <factoryAdapter type="Common.Logging.log4net.log4netLoggerFactoryAdapter, Common.Logging.Log4net1211">
                        <arg key="configType" value="INLINE" />
                    </factoryAdapter>
                </logging>
            </common>
            <log4net>
                <appender name="Alllogger" type="log4net.Appender.RollingFileAppender">
                    <file value="log4net-inline\" />
                    <datePattern value="yyyy-MM-dd\\'test.log'" />
                    <appendToFile value="true" />
                    <rollingStyle value="Composite" />
                    <maxSizeRollBackups value="10" />
                    <maximumFileSize value="100KB" />
                    <staticLogFileName value="false" />
                    <layout type="log4net.Layout.PatternLayout">
                        <conversionPattern value="%date [%thread] %-5level %logger [%property{NDC}] - %message%newline" />
                    </layout>
                </appender>
                <root>
                    <level value="ALL" />
                    <appender-ref ref="Alllogger" />
                </root>
            </log4net>
        </configuration>
        ```

## 程式碼加上 log

1.  引用 `Common.Logging`

    ```cs
    using Common.Logging;
    ```

2.  定義 logger

    ```cs
    private static ILog logger = LogManager.GetCurrentClassLogger();
    ```

3.  加上 log

    ```cs
    logger.Debug($"yowko-TestLog-{DateTime.Now}");
    ```

    *  關於 log level

        |Common.Logging.LogLevel|System.Diagnostics. TraceEventType|
        |--- |--- |
        |Trace|Verbose|
        |Debug|Verbose|
        |Error|Error|
        |Fatal|Critical|
        |Info|Information|
        |Warn|Warning|

## 使用單獨 log config

> configType 為
> 
> *   FILE：使用單獨的 config 檔
> *   FILE-WATCH：使用單獨 config 檔並監控變化

*   加上 configFile 屬性，並指定 config 檔案位置
*   以 NLog 為例 (2017/07/14 NLog 最新版為 4.1.2)

    *   web.config

        > 測試下來 NLog 只支援 `FILE` 不支援 `FILE-WATCH`

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <common>
                <logging>
                <factoryAdapter type="Common.Logging.NLog.NLogLoggerFactoryAdapter, Common.Logging.NLog41">
                    <arg key="configType" value="FILE" />
                    <arg key="configFile" value="~/NLog.config" />
                </factoryAdapter>
                </logging>
            </common>
        </configuration>
        ```
    *   NLog.config

        ```xml
        <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd" autoReload="true" throwExceptions="false" internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
            <targets async="true">
                <target xsi:type="File" name="f" fileName="${basedir}/logs-file/${shortdate}.log" layout="${longdate} ${uppercase:${level}} ${message}" />
            </targets>
            <rules>
                <logger name="*" minlevel="Debug" writeTo="f" />
            </rules>
        </nlog>
        ```
*   以 log4net 為例 (2017/07/14 log4net 最新版為 1.2.11)
    *   web.config

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <common>
                <logging>
                    <factoryAdapter type="Common.Logging.log4net.log4netLoggerFactoryAdapter, Common.Logging.Log4net1211">
                        <arg key="configType" value="FILE-WATCH" />
                        <arg key="configFile" value="~/log4net.config" />
                    </factoryAdapter>
                </logging>
            </common>
        </configuration>
        ```
    *   log4net.config

        ```xml
        <log4net>
            <appender name="Alllogger" type="log4net.Appender.RollingFileAppender">
                <file value="log4net-file\" />
                <datePattern value="yyyy-MM-dd\\'test.log'" />
                <appendToFile value="true" />
                <rollingStyle value="Composite" />
                <maxSizeRollBackups value="10" />
                <maximumFileSize value="100KB" />
                <staticLogFileName value="false" />
                <layout type="log4net.Layout.PatternLayout">
                <conversionPattern value="%date [%thread] %-5level %logger [%property{NDC}] - %message%newline" />
                </layout>
            </appender>
            <root>
                <!--DEBUG OR INFO OR ERROR OR WARN OR ALL-->
                <level value="ALL" />
                <appender-ref ref="Alllogger" />
            </root>
        </log4net>
        ```

## 心得

透過使用 Common.Logging 讓團隊中使用一致的 log 語法，又可以保留不同專案間使用不同 logging framework 的彈性，感覺還不賴，但還是有些缺點：Common.Logging 設定比較繁瑣些，需要注意的地方較多

依成果來看，選擇使用 Common.Logging 是明智之舉，避免了客製時需要針對不同 logging framework 客製兩套的狀況，也可以直接統一 log 寫法，甚至讓更換 logging framework 都變簡單了

# 參考資訊

1.  [Common-logging](http://net-commons.github.io/common-logging/)
2.  [net-commons/common-logging](https://github.com/net-commons/common-logging)
3.  [使用 ConfigurationSection 自訂 ASP.NET config (web.config) 區段](/2017/02/use-configurationsection-customize-aspnet-config.html)
4.  [Chapter 1. Common Logging](http://netcommon.sourceforge.net/docs/2.0.0/reference/html/ch01.html)
