---
title: "關於提昇 NLog 寫入檔案效能"
date: 2017-09-27T00:37:00+08:00
lastmod: 2021-11-02T00:37:19+08:00
draft: false
tags: ["NLog"]
slug: "nlog-async-keepfileopen"
aliases:
    - /2017/09/nlog-async-keepfileopen.html
---
## 關於提昇 NLog 寫入檔案效能

這是之前一直想做的測試筆記用來比較紀錄 NLog 參數設定對於寫入 log 的效能差異比較，起因是曾經在專案中遇到程式本身沒有問題，卻因為交易龐大產生大量 log 讓 nlog 頻繁讀寫檔案，加上使用傳統硬碟 disk io 速度受限，最後造成系統效能低落，形成嚴重效能瓶頸

剛好同事最近問到相關問題，就來紀錄一下相關比較數據吧

## 基本設定

1. 使用 console project
2. 安裝 NLog 及 NLog.config

    ![0nlognuget](https://user-images.githubusercontent.com/3851540/30871671-360ab8b0-a31a-11e7-881f-e7b146162ebc.png)

3. 加入寫 log 程式
    * 定義 logger

        ```cs
        private static ILogger logger = LogManager.GetCurrentClassLogger();
        ```

    * 執行 log

        ```cs
        static void Main(string[] args)
        {
            int logcount = 1000000;
            for (int i = 0; i < 5; i++)
            {
                LogToFile(i,logcount);
            }
                    }
                    private static void LogToFile(int index,int logcount)
        {
                        Stopwatch sw = new Stopwatch();
            sw.Start();
            for (int i = 0; i < logcount; i++)
            {
                logger.Debug($"{index}_{i}_{DateTime.Now}");
            }
            sw.Stop();
            logger.Info($"{index}_log 筆數：{logcount};總耗時毫秒：{sw.ElapsedMilliseconds}");
            LogManager.Flush();
        }
        ```

## 預設設定

> 使用原始 NLog.config 所提供的預設設定(僅調整 log folder 跟 layout)

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
    autoReload="true"
    throwExceptions="false"
    internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
    <targets>
        <target xsi:type="File" name="f" fileName="${basedir}/logs/origin/${shortdate}.${uppercase:${level}}.log"
                layout="${longdate} ${message}" />
    </targets>
    <rules>
        <logger name="*" minlevel="Debug" writeTo="f" />
    </rules>
</nlog>
```

![1_origin1000](https://user-images.githubusercontent.com/3851540/30871630-17665f22-a31a-11e7-92d1-588395f07b36.png)

![2origin100000](https://user-images.githubusercontent.com/3851540/30871633-176aafb4-a31a-11e7-91a6-c5f091b7d2c8.png)

|次數|1,000 筆|100,000 筆|
|--- |--- |--- |
|第一次|294|24113|
|第二次|265|26183|
|第三次|267|28779|
|第四次|212|30878|
|第五次|212|32353|
|平均|250|28461.2|

## 使用 keepFileOpen

> 在 `target` 中加上 `keepFileOpen="true"` 讓 log file 保持開啟，可以節省開檔、關檔的成本

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
    <targets>
        <target xsi:type="File" name="f" fileName="${basedir}/logs/keepFileOpen/${shortdate}.${uppercase:${level}}.log"
                layout="${longdate} ${message}" keepFileOpen="true"/>
    </targets>
    <rules>
        <logger name="*" minlevel="Debug" writeTo="f" />
    </rules>
</nlog>
```

![3fileopen1000](https://user-images.githubusercontent.com/3851540/30871632-17681254-a31a-11e7-985a-6a4348d84517.png))

![4fileopen100000](https://user-images.githubusercontent.com/3851540/30871634-1771fed6-a31a-11e7-863c-32425114b0ef.png)

|次數|1,000 筆|100,000 筆|
|--- |--- |--- |
|第一次|35|724|
|第二次|6|649|
|第三次|9|649|
|第四次|6|652|
|第五次|6|662|
|平均|12.4|667.2|

## 使用 async

> 在 `targets` 設定使用 asynchronous，讓 nlog 另起 thread 來寫入 log

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
    <targets async="true">
        <target xsi:type="File" name="f" fileName="${basedir}/logs/async/${shortdate}.${uppercase:${level}}.log"
                layout="${longdate} ${message}" />
    </targets>
    <rules>
        <logger name="*" minlevel="Debug" writeTo="f" />
    </rules>
</nlog>
```

![5async1000](https://user-images.githubusercontent.com/3851540/30871635-177c873e-a31a-11e7-9121-9335a63be14a.png)

![6async100000](https://user-images.githubusercontent.com/3851540/30871631-17676e9e-a31a-11e7-94d2-b7a75635cba1.png)

![9async1m](https://user-images.githubusercontent.com/3851540/30871637-17978dfe-a31a-11e7-8be8-b0d3034cfe6d.png)

|次數|1,000 筆|100,000 筆|1,000,000 筆|
|--- |--- |--- |--- |
|第一次|6|258|2100|
|第二次|1|207|2002|
|第三次|1|196|2007|
|第四次|1|189|1987|
|第五次|2|193|1993|
|平均|2.2|208.6|2017.8|

## 使用 async + keepFileOpen

> 同時使用 `async` 及 `keepFileOpen`

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
    <targets async="true">
        <target xsi:type="File" name="f" fileName="${basedir}/logs/asynckeepFileOpen/${shortdate}.${uppercase:${level}}.log"
                layout="${longdate} ${message}" keepFileOpen="true"/>
    </targets>
    <rules>
        <logger name="*" minlevel="Debug" writeTo="f" />
    </rules>
</nlog>
```

![7asynfile1000](https://user-images.githubusercontent.com/3851540/30871639-17b924be-a31a-11e7-8b61-3067092dda57.png)

![8asyncfile100000](https://user-images.githubusercontent.com/3851540/30871640-17ca5b8a-a31a-11e7-9495-e29f0d47888c.png)

![10asyncfile1M](https://user-images.githubusercontent.com/3851540/30871636-1796264e-a31a-11e7-80db-a7cfd051e121.png)

|次數|1,000 筆|100,000 筆|1,000,000 筆|
|--- |--- |--- |--- |
|第一次|6|264|2073|
|第二次|2|223|1963|
|第三次|1|218|1963|
|第四次|1|196|1957|
|第五次|1|197|1979|
|平均|2.2|219.6|1987|

## 總和比較

|設定|1,000 筆|100,000 筆|1,000,000 筆|
|--- |--- |--- |--- |
|預設|250|28461.2|-|
|keepFileOpen|12.4|667.2|-|
|async|2.2|208.6|2017.8|
|async + keepFileOpen|2.2|219.6|1987|

## 心得

除了預設設定之外，在少量 log (1000筆) 的情境下，各個設定的耗時差距都很小，但隨著 log 量增加至 10 萬筆時就會發現 async 效能比 keepFileOpen 較快，此時同時使用 async + keepFileOpen 效能上反而沒有單用 async 快，直到筆數來到百萬筆時，同時使用 async + keepFileOpen 效能上就會超越單用 async，但效果並不是非常顯著

另外有個普遍現象是，第一筆 log 的時間都要較久，原因是第一筆 log 包含建立檔案的時間

最後有個需要特別留意的是開檔效能，使用預設設定需要重複開檔、讀檔、寫檔，因此隨著檔案變大，寫入 log 的時間也就愈來愈長，相同的問題在其他設定上則未發生

## 參考資訊

1. [NLog Configuration file](https://github.com/NLog/NLog/wiki/Configuration-file)
2. [Should NLog flush all queued messages in the AsyncTargetWrapper when Flush() is called?](https://stackoverflow.com/questions/10492720/should-nlog-flush-all-queued-messages-in-the-asynctargetwrapper-when-flush-is)
