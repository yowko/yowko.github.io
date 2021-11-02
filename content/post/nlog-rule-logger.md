---
title: "NLog 設定 Rule 僅包含部份 Logger"
date: 2018-01-22T02:07:00+08:00
lastmod: 2021-11-02T02:07:15+08:00
draft: false
tags: ["套件","NLog"]
slug: "nlog-rule-logger"
aliases:
    - /2018/01/nlog-rule-logger.html
---
## NLog 設定 Rule 僅包含部份 Logger

log 是系統正式上線後，少數可以用來協助 debug 的資訊，而 debug 的難易度與解決問題的速度也就跟著 log 的品質而有極大的差異。

今天想要紀錄最近專案遇到的一個需求：某些 log 內容很少使用加上資料量又大實際效用不高，但卻是追查 bug 的最後一道不可或缺的防線(像是 EntityFramework 實際產出的 SQL script)，而其他 log 則紀錄了整個程式流程的相關資訊，所以打算將 db 的 script log 與其他程式執行資訊分開儲存，避免 db 的大量 log 形成雜訊而拖慢 debug 的速度，就來看看 nlog 可以如何設定吧

## 預設 nlog 設定

> 以下是 nlog.config 的設定範例

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">

  <variable name="myvar" value="myvalue"/>
  <targets>
    <target xsi:type="File" name="f" fileName="${basedir}/logs/${shortdate}.log"
            layout="${longdate} ${uppercase:${level}} ${message}" />
  </targets>

  <rules>
    <logger name="*" minlevel="Debug" writeTo="f" />
  </rules>
</nlog>
```

* target 設定說明
  * 預設寫至 `File`
  * target 名稱為 `f`
  * 檔案路徑為 `${basedir}/logs/${shortdate}.log`
  * log 格式為 `${longdate} ${uppercase:${level}} ${message}`

* rule 設定說明
  * 接受的 logger name 為 `*` (全部)
  * log 會紀錄的 level 為 `Debug`
  * log 寫入 name 為 `f` 的 target

## 如何僅包含指定 log

* 以需求來看很明確：在 rule 中正向表列 logger name

    > 但就 nlog 的文件看來是不支援這個需求，所以得要透過其他方式來達到目的了

1. 方法一：將排除的 log 指定 `final = "true"`
    * 個別儲存所有 log

        ```xml
        <logger name="db" minlevel="Debug" writeTo="db" />
        <logger name="audit" minlevel="Debug" writeTo="audit" />
        <logger name="normal" minlevel="Debug" writeTo="normal" />
        ```

    * 將需要結合的 log 組合

        ```xml
        <logger name="*" minlevel="Debug" writeTo="f" />
        ```

    * 指定排除不需結合的 log：加上 `final`

        ```xml
        <logger name="db" minlevel="Debug" writeTo="db" final="true"/>
        ```

    * 測試程式

        ```cs
        private static ILogger dblogger = LogManager.GetLogger("db");
        private static ILogger auditlogger = LogManager.GetLogger("audit");
        private static ILogger normallogger = LogManager.GetLogger("normal");
        static void Main(string[] args)
        {
            dblogger.Debug("db");
            auditlogger.Debug("audit");
            normallogger.Debug("normal");
        }
        ```

    * 實際效果
        * 個別 log 及 整合 log

            ![1logfiles](https://user-images.githubusercontent.com/3851540/35197288-14d6fc8a-ff18-11e7-8ccb-0bfeda5627ab.png)

        * db log 未寫至 all log 中

            ![2partlog](https://user-images.githubusercontent.com/3851540/35197289-14ffd3e4-ff18-11e7-88c6-af02d65c5250.png)

    * 完整 nlog.config

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
            autoReload="true"
            throwExceptions="false"
            internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
                    <variable name="myvar" value="myvalue"/>
        <targets>
            <target xsi:type="File" name="db" fileName="${basedir}/logs/db/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
            <target xsi:type="File" name="audit" fileName="${basedir}/logs/audit/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
            <target xsi:type="File" name="normal" fileName="${basedir}/logs/normal/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
                        <target xsi:type="File" name="f" fileName="${basedir}/logs/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
        </targets>
                    <rules>
            <logger name="db" minlevel="Debug" writeTo="db" final="true"/>
            <logger name="audit" minlevel="Debug" writeTo="audit" />
            <logger name="normal" minlevel="Debug" writeTo="normal" />
            <logger name="*" minlevel="Debug" writeTo="f" />
        </rules>
        </nlog>
        ```

2. 方法二：調整 logger 名稱
    * 個別儲存所有 log

        ```xml
        <logger name="db" minlevel="Debug" writeTo="db" />
        <logger name="main.audit" minlevel="Debug" writeTo="audit" />
        <logger name="main.normal" minlevel="Debug" writeTo="normal" />
        ```

    * 透過 logger name 將需要結合的 log 組合

        ```xml
        <logger name="main.*" minlevel="Debug" />
        ```

    * 測試程式

        ```cs
        private static ILogger dblogger = LogManager.GetLogger("db");
        private static ILogger auditlogger = LogManager.GetLogger("main.audit");
        private static ILogger normallogger = LogManager.GetLogger("main.normal");
        static void Main(string[] args)
        {
            dblogger.Debug("db");
            auditlogger.Debug("audit");
            normallogger.Debug("normal");
        }
        ```

    * 實際效果

        * 個別 log 及 整合 log

            ![3loggername](https://user-images.githubusercontent.com/3851540/35197290-15288c9e-ff18-11e7-9db3-30936fde6f85.png)

        * db log 未寫至 all log 中

            ![4partlog](https://user-images.githubusercontent.com/3851540/35197291-15738082-ff18-11e7-903c-6b2fecedc046.png)

    * 完整 nlog.config

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
            autoReload="true"
            throwExceptions="false"
            internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
                    <variable name="myvar" value="myvalue"/>
        <targets>
            <target xsi:type="File" name="db" fileName="${basedir}/logs/db/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
            <target xsi:type="File" name="audit" fileName="${basedir}/logs/audit/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
            <target xsi:type="File" name="normal" fileName="${basedir}/logs/normal/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
                        <target xsi:type="File" name="f" fileName="${basedir}/logs/${shortdate}.log"
                    layout="${longdate} ${uppercase:${level}} ${message}" />
        </targets>
                    <rules>
            <logger name="db" minlevel="Debug" writeTo="db" />
            <logger name="main.audit" minlevel="Debug" writeTo="audit" />
            <logger name="main.normal" minlevel="Debug" writeTo="normal" />
            <logger name="main.*" minlevel="Debug" writeTo="f" />
        </rules>
        </nlog>
        ```

## 心得

雖然 nlog 要達成正向表面 log 對象 的做法比較沒那麼直覺(或者只是沒查到正確的文件)，但其他設定上相較其他 log library 還是方便不少

趁著專案剛好有相關需求紀錄一下現在的做法，也許過陣子或是新版就會有不同的設定方式可更簡易地符合需求了

## 參考資訊

1. [NLog Configuration-file - Rules](https://github.com/NLog/NLog/wiki/Configuration-file#rules)
