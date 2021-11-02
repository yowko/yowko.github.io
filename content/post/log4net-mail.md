---
title: "讓 log4net 收到指定錯誤 Level 發送 mail"
date: 2018-05-27T02:02:00+08:00
lastmod: 2021-11-02T02:02:22+08:00
draft: false
tags: ["套件","csharp"]
slug: "log4net-mail"
aliases:
    - /2018/05/log4net-mail.html
---
## 讓 log4net 收到指定錯誤 Level 發送 mail

平常我自己本身慣用的 log 套件是 nlog，主要原因是因為設定相對較簡潔，加上多年前看過的效能比較 - [Benchmarking 5 popular .NET logging libraries](https://www.loggly.com/blog/benchmarking-5-popular-net-logging-libraries/) 結果是 nlog 效能較好，不過前陣子看到另一篇文章 - [.Netcore之日誌組件Log4net、Nlog性能比較](http://www.itread01.com/articles/1505163610.html) 重新比較 nlog 與 log4net 的效能則得出 log4net 效能較好的結論，所以針對兩者效能問題我想自己動手比較看看，再下結論

在取得效能比較結論前之前，因緣際會下需要調整其他同事的專案，剛好是使用 log4net，所以透過這個機會紀錄一下使用方式

BTW：原本想要大致紀錄一下 log4net 的相關設定及用法，但文件愈看愈多卻也愈搞不清楚設定細節，決定改天有需要時再另外紀錄，今天就只紀錄如何讓 log4net 發出 mail 囉

## 基本設定

> 以下範例會將 log4net 設定與 application config (app.config 或是 web.config) 放在一起

1. 加入 log4net 的 configSection

    > 讓程式可以利用指定的 log4net handler 來處理 log4net config 區塊

    ```xml
    <configSections>
        <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
    </configSections>
    ```

2. 加入 log4net 相關設定

    > log4net 區段的組成內容

    - appender

        > 允許零個或多個，用來設定 log 輸出的實際對象：file、mail、console...etc，實際輸出訊息的格式也設定於此
    - logger

        > 允許零個或多個，預設會繼承 root 的參數設定，程式指定 logger name，再經由 logger 設定分派給 appender 處理
    - renderer

        >允許零個或多個，透過實作 log4net.ObjectRenderer.IObjectRenerer 來客製化 log 輸出
    - root

        > 允許零個或一個，logger 最上層的參數，會被所有 logger 繼承
    - param

        > 允許零個或多個，用來指定 log4net 全域用參數

    ```xml
    <log4net>
        <appender name="SmtpAppender" type="log4net.Appender.SmtpAppender">
            <to value="yowko@yowko.com" />
            <from value="yowko@yowko.com" />
            <subject value="test logging message" />
            <smtpHost value="127.0.0.1" />
            <bufferSize value="512" />
            <lossy value="false" />
            <layout type="log4net.Layout.PatternLayout">
                <conversionPattern value="%newline%date [%thread] %-5level %logger [%property{NDC}] - %message%newline%newline%newline" />
            </layout>
        </appender>
        <logger name="UserQuery">
            <level value="DEBUG" />
            <appender-ref ref="SmtpAppender" />
        </logger>
    </log4net>
    ```

3. 實際使用

    ```cs
    //建立 logger
    var log = LogManager.GetLogger(this.GetType());
    //讀取 log4net 相關設定
    XmlConfigurator.Configure();
    // 開始寫 log
    log.Debug("Test Debug");
    log.Info("Test Info");
    log.Warn("Test Warn");
    log.Error("Test Error");
    log.Fatal("Test Fatal");
    ```

## log 設定說明

1. logger
    - `name` 屬性

        > 需與程式中所指定的 logger 相同

        - 手動指定 logger 名稱

            ```cs
            var log = LogManager.GetLogger("UserQuery");
            ```  

        - 使用當前 class 的名稱
            - method 1 ：

                ```cs
                var log = LogManager.GetLogger(this.GetType());
                ```  

            - method 2：

                ```cs
                var log = log4net.LogManager.GetLogger(System.Reflection.MethodBase.GetCurrentMethod().DeclaringType);
                ```

    - `level` 元素

        > 指定最低 log 等級 ，會覆寫 `root` 設定
    - `appender-ref` 元素

        > 指定使用的 appender
2. SmtpAppender

    > `to`, `from`, `subject` 與 `smtpHost` 是必要條件

    - `to`

        > mail 對象
    - `from`

        > 送出的 mail
    - `subject`

        > mail 主旨
    - `smtpHost`

        > mail server 位址
    - `bufferSize`

        > 緩衝區大小
    - `lossy`

        > 用來設定是否可以允許資料遺失，只在繼承自 `BufferingAppenderSkeleton` 的 appender 出現，SmtpAppender 是其一，預設是 `false` (避免資料不見); 如設為 `true` 需搭配 `evaluator` 設定，緩衝區滿時會將最舊的 log 刪除
        - 使用預設值 --> 要特別注意，這樣的設定不會立即發送，需待 bufferSize 滿了才統一發送，可以將 bufferSize 設為 `1` 即會立即發送

            ```xml
            <log4net>
                <appender name="SmtpAppender" type="log4net.Appender.SmtpAppender">
                <to value="yowko@yowko.com" />
                <from value="yowko@yowko.com" />
                <subject value="fail message" />
                <smtpHost value="127.0.0.1" />
                <bufferSize value="512" />
                <layout type="log4net.Layout.PatternLayout">
                    <conversionPattern value="%newline%date [%thread] %-5level %logger [%property{NDC}] - %message%newline%newline%newline" />
                </layout>
                </appender>
                <logger name="UserQuery">
                    <level value="DEBUG" />
                    <appender-ref ref="SmtpAppender" />
                </logger>
            </log4net>
            ```

        - 結果

            ![2lossyfalse](https://user-images.githubusercontent.com/3851540/40578964-2f6a304a-6150-11e8-98e7-a72449483d2d.png)
    - `evaluator`
         - `threshold`

            > 在 `threshold` 指定觸發發送 mail 的 level (含以上 level)

        > 低於 `threshold` 指定 level 的 log 都會先存在緩衝區，直到指定 level 事件出現在一併將緩衝區的 log 輸出

        - 實例設定

            ```xml
            <log4net>
                <appender name="SmtpAppender" type="log4net.Appender.SmtpAppender">
                <to value="yowko@yowko.com" />
                <from value="yowko@yowko.com" />
                <subject value="fail message" />
                <smtpHost value="127.0.0.1" />
                <bufferSize value="512" />
                <lossy value="true" />
                <evaluator type="log4net.Core.LevelEvaluator">
                    <threshold value="ERROR" />
                </evaluator>
                <layout type="log4net.Layout.PatternLayout">
                    <conversionPattern value="%newline%date [%thread] %-5level %logger [%property{NDC}] - %message%newline%newline%newline" />
                </layout>
                </appender>
                <logger name="UserQuery">
                    <level value="DEBUG" />
                    <appender-ref ref="SmtpAppender" />
                </logger>
            </log4net>
            ```

        - 結果

            ![1evaluatorthehold](https://user-images.githubusercontent.com/3851540/40578963-2f26e524-6150-11e8-98db-a067a820aa9f.png)

## 如何限定只處理特定 level

1. 方法一： 透過 `filter` 將 `levelMin` 與 `levelMax` 都指定為 `ERROR`

    ```xml
    <log4net>
        <appender name="SmtpAppender" type="log4net.Appender.SmtpAppender">
        <to value="yowko@yowko.com" />
        <from value="yowko@yowko.com" />
        <subject value="fail message" />
        <smtpHost value="127.0.0.1" />
        <bufferSize value="512" />
        <lossy value="true" />
        <evaluator type="log4net.Core.LevelEvaluator">
            <threshold value="ERROR" />
        </evaluator>
        <filter type="log4net.Filter.LevelRangeFilter">
            <levelMin value="ERROR" />
            <levelMax value="ERROR" />
        </filter>
        <layout type="log4net.Layout.PatternLayout">
            <conversionPattern value="%newline%date [%thread] %-5level %logger [%property{NDC}] - %message%newline%newline%newline" />
        </layout>
        </appender>
        <logger name="UserQuery">
        <level value="DEBUG" />
        <appender-ref ref="SmtpAppender" />
        </logger>
    </log4net>
    ```

2. 方法二：透過 `filter` 將 `levelToMatch` 指定為 `ERROR` 並加上 `DenyAllFilter`

    ```xml
    <log4net>
        <appender name="SmtpAppender" type="log4net.Appender.SmtpAppender">
        <to value="yowko@yowko.com" />
        <from value="yowko@yowko.com" />
        <subject value="fail message" />
        <smtpHost value="127.0.0.1" />
        <bufferSize value="512" />
        <lossy value="true" />
        <evaluator type="log4net.Core.LevelEvaluator">
            <threshold value="ERROR" />
        </evaluator>
        <filter type="log4net.Filter.LevelMatchFilter,log4net">
            <levelToMatch value="ERROR" />
        </filter>
        <filter type="log4net.Filter.DenyAllFilter" />
        <layout type="log4net.Layout.PatternLayout">
            <conversionPattern value="%newline%date [%thread] %-5level %logger [%property{NDC}] - %message%newline%newline%newline" />
        </layout>
        </appender>
        <logger name="UserQuery">
            <level value="DEBUG" />
            <appender-ref ref="SmtpAppender" />
        </logger>
    </log4net>
    ```

## 心得

不得不說 log4net 還真不是我的菜呀，功能也許很強大很彈性，但是文件實在令人不敢恭維，想要達到特定功能貌似許多做法，但卻好像遶了一大圈：我只想在 ERROR 時寄送 mail ，level 包含 FATAL 都不經由 mail，我反覆測試了好多種組合才真正解決問題，或許是我資質駑鈍，雖然經過這次經驗後我日後應該可以快速想到做法，但如果這樣簡單的需求都要讓入門者花那麼多時間嘗試，我想這樣的使用者體驗並不合格

## 參考資訊

1. [SmtpAppender Class](http://logging.apache.org/log4net/release/sdk/html/T_log4net_Appender_SmtpAppender.htm)
2. [Apache log4net™ Config Examples](http://logging.apache.org/log4net/release/config-examples.html)
