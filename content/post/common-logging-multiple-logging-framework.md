---
title: "使用 Common.Logging 同時將 log 寫至多個 Logging Framework"
date: 2017-07-15T23:23:00+08:00
lastmod: 2020-12-11T23:23:00+08:00
draft: false
tags: ["套件","log4net","NLog"]
slug: "common-logging-multiple-logging-framework"
aliases:
    - /2017/07/common-logging-multiple-logging-framework.html
---
# 使用 Common.Logging 同時將 log 寫至多個 Logging Framework
之前文章 [使用 Common.Logging 搭配 NLog 及 Log4Net](/2017/07/common-logging.html) 介紹到透過 Common.Logging 可以使用一致的 log 語法將 log 轉由不同 logging framework 處理，在不同專案間就可以使用統一 log 語法而且依團隊習慣自由選擇不同的 logging framework

測試過程聯想到，是不是可以同時發送 log 給不同的 logging framework，動手測試果然不支援 XD，後來找到方法，順手紀錄一下

我聯想到的使用情境是不同的 logging framework 有不同的用途及目的，也許 layout 也不同，雖然大部份情境我個人會使用 NLog 的多個 target 來做，但多個可選方案也不錯，就來看看可以怎麼做

## 直接在 config 中使用多個 Logging Framework

> 直接就噴錯誤了

*   錯誤訊息內容

    ```
    Configuration Error
        
    Description: An error occurred during the processing of a configuration file required to service this request. Please review the specific error details below and modify your configuration file appropriately. 
        
    Parser Error Message: An error occurred creating the configuration section handler for common/logging: Only one <factoryAdapter> element allowed
    ```

*   錯誤訊息畫面

    ![1error](https://user-images.githubusercontent.com/3851540/28240376-7af86fc6-69b3-11e7-8d4a-8276a4b9b7e4.png)

    *   大意就是不支援多個 `factoryAdapter` 設定



## 安裝 Common.Logging.MultipleLogger 套件

*   目前到只有 Beta2 版本
*   使用 Package Manager Console 安裝範例

    ```
    Install-Package Common.Logging.MultipleLogger -Version 3.4.0-Beta2 -Pre
    ```

## 修改 config 設定

1.  加上 config section 定義

    ```
    <section name="logging.multipleLoggers" type="Common.Logging.MultipleLogger.ConfigurationSectionHandler, Common.Logging.MultipleLogger" />
    ```

    *   會同時需要兩組 section 設定如下

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <configSections>
                <sectionGroup name="common">
                    <section name="logging" type="Common.Logging.ConfigurationSectionHandler, Common.Logging" />
                    <section name="logging.multipleLoggers" type="Common.Logging.MultipleLogger.ConfigurationSectionHandler, Common.Logging.MultipleLogger" />
                </sectionGroup>
            </configSections>
        </configuration>
        ```

2.  同時啟用多個 logging framework section

    > 以 NLog 與 Log4Net 為例

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <configSections>
            <sectionGroup name="common">
                <section name="logging" type="Common.Logging.ConfigurationSectionHandler, Common.Logging" />
                <section name="logging.multipleLoggers" type="Common.Logging.MultipleLogger.ConfigurationSectionHandler, Common.Logging.MultipleLogger" />
            </sectionGroup>
            <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
            <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler,log4net" />
        </configSections>
    </configuration>
    ```

3.  factoryAdapter 使用 MultipleLogger

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <common>
            <logging>
                <factoryAdapter type="Common.Logging.MultipleLogger.MultiLoggerFactoryAdapter, Common.Logging.MultipleLogger">
                    <arg key="MeaninglessKey" value="MeaninglessValue" />
                </factoryAdapter>
            </logging>
        </common>
    </configuration>
    ```
4.  將 NLog 及 Log4Net 的 factoryAdapter 定義移至 `logging.multipleLoggers` 中

    > 這邊的 factoryAdapter type 記得要指定正確 logging framework 的版本

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <common>
            <logging.multipleLoggers>
                <factoryAdapter type="Common.Logging.Log4Net.Log4NetLoggerFactoryAdapter, Common.Logging.Log4net1211">
                    <arg key="configType" value="FILE-WATCH" />
                    <arg key="configFile" value="~/log4net.config" />
                </factoryAdapter>
                <factoryAdapter type="Common.Logging.NLog.NLogLoggerFactoryAdapter, Common.Logging.NLog41">
                    <arg key="configType" value="FILE" />
                    <arg key="configFile" value="~/NLog.config" />
                </factoryAdapter>
            </logging.multipleLoggers>
        </common>
    </configSection>
    ```

## 程式碼加上 log

> 這個部份都一樣不需要修改

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
## 心得

測試幾個方法一直不得其門而入，使用的套件： Common.Logging.MultipleLogger 可能是因為還在 beta 找不到相關文件，後來好不容易在 test 專案找到相關設定，有興趣的可以參考 [Common.Logging.MultipleLogger.Tests](https://github.com/net-commons/common-logging/blob/master/test/Common.Logging.MultipleLogger.Tests/App.config)，最後靠著測試專案的設定總算是搞定了 ~~~呼 還好人家有乖乖寫測試

# 參考資訊

1.  [Common.Logging.MultipleLogger.Tests](https://github.com/net-commons/common-logging/blob/master/test/Common.Logging.MultipleLogger.Tests/App.config)
2.  [ommon.Logging.MultipleLogger](https://www.nuget.org/packages/Common.Logging.MultipleLogger/)
3.  [Common-logging](https://net-commons.github.io/common-logging/)
