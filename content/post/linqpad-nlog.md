---
title: "在 LINQPad 中使用 NLog"
date: 2018-05-18T00:45:00+08:00
lastmod: 2018-10-06T11:17:03+08:00
draft: false
tags: ["NLog","Tools"]
slug: "linqpad-nlog"
aliases:
    - /2018/05/linqpad-nlog.html
---
# 在 LINQPad 中使用 NLog
LINQPad 的便利性我想不必我多提，而我自己常常在 LINQPad 上進行主要核心流程功能的 poc 開發，或是將特定功能從原系統抽離以加速功能驗證，今天遇到的情境也是如此，希望在不改動原本程式的主架構下略過其他不必要的系統驗證授權部份，完整保留原本的 log 行為與執行流程並且順利完成程式的調整，這樣一來功能完成也可以與原系統無縫接軌了，但現實常常與理想會有些距離，就來看看如何在 LINQPad 中正常使用 NLog 吧

## 方法一：使用程式設定 NLog config
- 建立 `NLogLINQPadExtensions`
    
    > 用來指定 NLognlog 相關設定 
    
    ```cs
    public static class NLogLINQPadExtensions
    {
        public static void ConfigureLogging()
        {
            //指定 config
            var nlogConfig = @"
                    <nlog>
                        <targets>
                        <target name='console' type='File' fileName='D:/Test/ConfigInCode/${shortdate}_${level}.log' layout='${longdate} | ${uppercase:${level}} | ${message}' />
                        </targets>
                        <rules>
                        <logger name='*' minlevel='Debug' writeTo='console' />
                        </rules>
                    </nlog>
            ";

            using (var sr = new StringReader(nlogConfig))//讀取 config string
            using (var xr = XmlReader.Create(sr))//使用 xml 方式讀取 config 
            {
                //將 config 設定給 nlog
                NLog.LogManager.Configuration = new XmlLoggingConfiguration(xr, null);
                //重新套用 nlog config
                NLog.LogManager.ReconfigExistingLoggers();
            }

        }
    }
    ```
- 實際使用
    
    ```cs
    //設定 config
	NLogLINQPadExtensions.ConfigureLogging();

	//使用指定名稱初始化 nlog
	var logger = LogManager.GetCurrentClassLogger();
	logger.Debug("Yowko Test");
    ``` 
- 完成程式碼
    
    ```cs
    void Main()
    {
        //設定 config
        NLogLINQPadExtensions.ConfigureLogging();

        //使用指定名稱初始化 nlog
        var logger = LogManager.GetCurrentClassLogger();
        logger.Debug("Yowko Test");
    }

    public static class NLogLINQPadExtensions
    {
        public static void ConfigureLogging()
        {
            //指定 config
            var nlogConfig = @"
                    <nlog>
                        <targets>
                        <target name='f' type='File' fileName='D:/Test/ConfigInCode/${shortdate}_${level}.log' layout='${longdate} | ${uppercase:${level}} | ${message}' />
                        </targets>
                        <rules>
                        <logger name='*' minlevel='Debug' writeTo='f' />
                        </rules>
                    </nlog>
            ";

            using (var sr = new StringReader(nlogConfig))//讀取 config string
            using (var xr = XmlReader.Create(sr))//使用 xml 方式讀取 config 
            {
                //將 config 設定給 nlog
                NLog.LogManager.Configuration = new XmlLoggingConfiguration(xr, null);
                //重新套用 nlog config
                NLog.LogManager.ReconfigExistingLoggers();
            }

        }
    }
    ``` 

## 方法二： 將 Nlog.config 跟 LINQPad.exe 放一起

> LINQPad.exe 預設位置為 `C:\Program Files (x86)\LINQPad5\`

1. 準備 `Nlog.config`
    
    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
        autoReload="true"
        throwExceptions="false"
        internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">
    <targets async="true">
        <target name="f" type="File" fileName="D:/Test/ConfigWithLINQPad/${shortdate}_${level}.log" layout="${longdate} | ${uppercase:${level}} | ${message}" />
    </targets>
    <rules>
        <logger name="*" minlevel="Debug" writeTo="f" />
    </rules>
    </nlog>
    ``` 
2. 將 `Nlog.exe` 與 LINQPad.exe 放在一起
    
    > 我實測下不需要重新啟動 `LINQPad`，但如果遲遲無法使用或許可以考慮重啟 LINQPad 試試
3. 實際使用 
    
    ```cs
    void Main()
    {
        var logger = LogManager.GetCurrentClassLogger();
        logger.Debug("Yowko Test");
    }
    ``` 

## 方法三： 透過 LINQPad 設定 nlog.config
1. 開啟 app.config
    
    > 空白處 按右鍵 -->  app.config

    ![1openappconfig](https://user-images.githubusercontent.com/3851540/40189133-37448e06-5a2e-11e8-9db7-24f69f326324.png)
2. 將 nlog.config 內容填入
    
    > 記得指定 `configSections`
    
    ![2editappconfig](https://user-images.githubusercontent.com/3851540/42794679-c53b6f8c-89b2-11e8-8153-50b5a3c2d3d2.png)
    
    ```xml
    <configuration>
        <configSections>
            <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
        </configSections>
        <nlog>
            <targets async="true">
                <target name="f" type="File" fileName="D:/Test/ConfigInLINQPad/${shortdate}_${level}.log" layout="${longdate} | ${uppercase:${level}} | ${message}" />
            </targets>
            <rules>
                <logger name="*" minlevel="Debug" writeTo="f" />
            </rules>
        </nlog>
    </configuration>
    ``` 
    
3. 實際使用
    
    ```cs
    void Main()
    {
        var logger = LogManager.GetCurrentClassLogger();
        logger.Debug("Yowko Test");
    }
    ```

## 心得
原本我有些排斥 `方法一：使用程式設定 nlog config`，認為與原有系統透過 config fiile 方式設定 nlog 的做法不同，但真的設定一次後覺得比起 `方法二： 將 Nlog.config 跟 LINQPad.exe 放一起` 方便許多，就連原本認為失真度最小的 `方法三： 透過 LINQPad 設定 nlog.config` 在修改的便利性也沒有 `方法一：使用程式設定 nlog config` 來得快速，加上 `方法二： 將 Nlog.config 跟 LINQPad.exe 放一起` 會影響所有 LINQPad 的執行行為很快就遭到排除了

不過整體說來 `方法一：使用程式設定 nlog config` 與 `方法三： 透過 LINQPad 設定 nlog.config` 差異並不大，可以依實際情況來決定使用方式：
- `方法一：使用程式設定 nlog config` 直接改 code 就能用
- `方法三： 透過 LINQPad 設定 nlog.config` 是確認異動後可以直接 copy 進原系統，各有勝負呀

# 參考資訊
1. [NLog via LINQPad - Where to put config file?](https://stackoverflow.com/questions/24024336/nlog-via-linqpad-where-to-put-config-file)
2. [yoeun/app.config.xml](https://gist.github.com/yoeun/3353036)