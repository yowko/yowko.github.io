---
title: "將 ASP.NET Core 的預設 log 輸出至 NLog 或 Serilog"
date: 2019-03-10T21:00:00+08:00
lastmod: 2019-03-10T21:00:31+08:00
draft: false
tags: ["ASP.NET Core","Log","Nlog","Serilog"]
slug: "asp-net-core-default-log-nlog-serilog"
---
# 將 ASP.NET Core 的預設 log 輸出至 NLog 或 Serilog
ASP.NET Core 預設會將 WebHost 執行細節透過 stdout 輸出至 console 上，application 的所有動作都一目暸然，這在 debug 時很方便，不過部署在一般的 server 就不會有人時時刻刻去盯著 console，所以需要將 log 內容透過其他方式(e.g. 檔案、db、...etc)紀錄下來備查

過去我個人是比較習慣 NLog 的設定風格，而現在團隊使用 Serilog，所以一併紀錄兩個 log framework 的設定方式


## 基本環境說明
1. .NET Core 2.2.101
2. ASP.NET Core 2.2.0
3. 使用 ASP.NET Core - API 預設範本建立環境
4. NLog.Web.AspNetCore 4.8.0
5. Serilog.AspNetCore 2.1.1
6. Serilog.Sinks.File 4.0.0
7. Serilog.Settings.Configuration 3.0.1

## 預設行為：輸出至 console

```
Hosting environment: Development
Content root path: C:\Users\yowko\source\repos\TestLog\LogTest
Now listening on: https://localhost:5001
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET https://localhost:5001/api/values
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'LogTest.Controllers.ValuesController.Get (LogTest)'
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Route matched with {action = "Get", controller = "Values"}. Executing action LogTest.Controllers.ValuesController.Get (LogTest)
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method LogTest.Controllers.ValuesController.Get (LogTest) - Validation state: Valid
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action method LogTest.Controllers.ValuesController.Get (LogTest), returned result Microsoft.AspNetCore.Mvc.ObjectResult in 0.3302ms.
info: Microsoft.AspNetCore.Mvc.Infrastructure.ObjectResultExecutor[1]
      Executing ObjectResult, writing value of type 'System.String[]'.
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action LogTest.Controllers.ValuesController.Get (LogTest) in 179.3543ms
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'LogTest.Controllers.ValuesController.Get (LogTest)'
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 256.8033ms 200 application/json; charset=utf-8
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET https://localhost:5001/robots.txt?1552204068727
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 5.5067ms 404
```

![1default](https://user-images.githubusercontent.com/3851540/54085483-a92e5300-4379-11e9-9bfc-9f312f551215.png)

## NLog
1. 安裝 NLog.Web.AspNetCore

    - Package Manager

        ```
        Install-Package NLog.Web.AspNetCore
        ```
    - .NET CLI

        ```
        dotnet add package NLog.Web.AspNetCore
        ```
2. 建立 `nlog.config` 檔案

    > 位置為專案根目錄下

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        autoReload="true"
        internalLogLevel="Info"
        internalLogFile="c:\temp\internal-nlog.txt">

    <!-- 啟用 ASP.NET Core layout renderers -->
    <extensions>
        <add assembly="NLog.Web.AspNetCore"/>
    </extensions>

    <!-- log 儲存目標 -->
    <targets>
        <!-- 儲存目標類型為 "檔案"  -->
        <target xsi:type="File" name="allfile" fileName=".\logs\nlog-all-${shortdate}.log"
                layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />

        <!-- 儲存目標類型為 "檔案", only own logs.並使用部份 ASP.NET core renderers 的內容 -->
        <target xsi:type="File" name="ownFile-web" fileName=".\logs\nlog-own-${shortdate}.log"
                layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}|url: ${aspnet-request-url}|action: ${aspnet-mvc-action}" />
    </targets>

    <!-- 設定 logger 名稱與 log 儲存目標的對應 -->
    <rules>
        <!-- 所有 log -->
        <logger name="*" minlevel="Trace" writeTo="allfile" />

        <!-- 將來自於 Microsoft. assembly 的 Info 以下 (Info,Debug,Trace) log 都排除 (沒有 writeTo 就不會輸出 )-->
        <!-- <logger name="Microsoft.*" maxlevel="Info" final="true" />-->
        <logger name="*" minlevel="Trace" writeTo="ownFile-web" />
    </rules>
    </nlog>
    ```
3. 將 `nlog.config` 設定輸出

    - nlog.config 上按右鍵 --> Properties

        ![2properties](https://user-images.githubusercontent.com/3851540/54085484-a9c6e980-4379-11e9-86ff-551491722f71.png)
    - Copy to Output Directory --> Copy always

        ![3copyalways](https://user-images.githubusercontent.com/3851540/54085485-a9c6e980-4379-11e9-982d-9d13ca3f157d.png)
4. 在 program.cs 中設定啟用 NLog

    ```cs
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                //.ConfigureLogging(logging =>
                //{
                //    //清除其他 logger provide 如預設 console (一經設定 log 就不會在 console 上出現)
                //    logging.ClearProviders();
                //    //設定最低 log 輸出 level
                //    logging.SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Trace);
                //})
                .UseNLog();
    }
    ```
5. 完整 program.cs 內容

    ```cs
    using Microsoft.AspNetCore;
    using Microsoft.AspNetCore.Hosting;
    using NLog.Web;
    using Microsoft.Extensions.Logging;

    namespace LogTest
    {
        public class Program
        {
            public static void Main(string[] args)
            {
                CreateWebHostBuilder(args).Build().Run();
            }

            public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
                WebHost.CreateDefaultBuilder(args)
                    .UseStartup<Startup>()
                    //.ConfigureLogging(logging =>
                    //{
                    //    //清除其他 logger provide：預設 console
                    //    logging.ClearProviders();
                    //    //設定最底 log 輸出 level
                    //    logging.SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Trace);
                    //})
                    .UseNLog();
        }
    }
    ```

## Serilog

1. 安裝基礎套件 Serilog.AspNetCore 

    - Package Manager

        ```
        Install-Package Serilog.AspNetCore
        ```
    - .NET CLI

        ```
        dotnet add package Serilog.AspNetCore
        ```
2. 視 log 儲存類型安裝套件(以 file 為例)：Serilog.Sinks.File

    - Package Manage

        ```
        Install-Package Serilog.Sinks.File
        ```
    - .NET CLI

        ```
        dotnet add package Serilog.Sinks.File
        ```
3. 安裝 json config 用套件 : Serilog.Settings.Configuration

    > xml config 需要安裝 Serilog.Settings.AppSettings

    - Package Manage

        ```
        Install-Package Serilog.Settings.Configuration
        ```
    - .NET CLI

        ```
        dotnet add package Serilog.Settings.Configuration
        ```
4. 設定 Serilog 相關參數

    > 設定值寫至 `appsettings.json`

    ```json
    {
    "Serilog": {
        "Using": [  "Serilog.Sinks.File" ],
        "MinimumLevel": "Debug",
        "WriteTo": [
                {
                    "Name": "File",
                    "Args": {
                    "path": ".\\Serilogs\\serilog-configuration-sample.txt",
                    "outputTemplate": "{Timestamp:o} [{Level:u3}] ({Application}/{MachineName}/{ThreadId}) {Message}{NewLine}{Exception}",
                    "rollingInterval": "Day"
                    }
                }
            ]
        }
    }
    ```
5. 設定啟用 Serilog
    - Main method
    
        ```cs
        //從 appsettings.json 讀取設定資料
        var configuration = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile(path: "appsettings.json", optional: false, reloadOnChange: true)
                .Build();
        
        //使用從 appsettings.json 讀取到的內容來設定 logger
        Log.Logger = new LoggerConfiguration()
                    .ReadFrom.Configuration(configuration)
                .CreateLogger();
        ```
    - CreateWebHostBuilder method

        ```cs
        WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                //註冊 Serilog
                .UseSerilog();
        ```
6. programs.cs 完整內容

    ```cs
    using Microsoft.AspNetCore;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Extensions.Configuration;
    using Serilog;
    using System.IO;

    namespace LogTest
    {
        public class Program
        {
            public static void Main(string[] args)
            {
                var configuration = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile(path: "appsettings.json", optional: false, reloadOnChange: true)
                .Build();

                Log.Logger = new LoggerConfiguration()
                    .ReadFrom.Configuration(configuration)
                .CreateLogger();


                CreateWebHostBuilder(args).Build().Run();
            }

            public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
                WebHost.CreateDefaultBuilder(args)
                    .UseStartup<Startup>()
                    .UseSerilog();
        }
    }
    ```

# 心得

NLog 有點像過去的 .NET Framework 中的 System.Web ：功能齊全，只要一經引用幾乎沒什麼做不到的;Serilog 則像 .NET Core，多數功能都能滿足，但功能拆分細微，需要什麼就單獨安裝、升級

兩者相較下 NLog 感覺是相對直覺、或許該說是習慣，但就文件的完整性來說 NLog 則是佔絕對優勢，同樣的功能 NLog 可以直接從 wiki 上找到設定方式，但 Serilog 就得要東翻西找，加上還參雜了不同版本的 ASP.NET Core 內容造成無法馬上辨別出是否為正確做法

整個 Serilog 在使用的概念上與文件是比較接近 .NET Core，好處、壞處都很像XD

# 參考資訊
1. [NLog](https://nlog-project.org/)
2. [Getting started with ASP.NET Core 2](https://github.com/NLog/NLog.Web/wiki/Getting-started-with-ASP.NET-Core-2)
3. [serilog/serilog](https://github.com/serilog/serilog/wiki/Getting-Started)
4. [Exploring Serilog v2 - Lets Go!](http://merbla.com/2018/04/02/exploring-serilog-v2---lets-go/)
5. [NLog vs log4net vs Serilog: Compare .NET Logging Frameworks](https://stackify.com/nlog-vs-log4net-vs-serilog/)