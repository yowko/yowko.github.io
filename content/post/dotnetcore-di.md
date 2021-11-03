---
title: "在 .NET Core console 上使用 Dependency Injection - DI"
date: 2018-11-28T23:45:00+08:00
lastmod: 2021-11-03T23:44:30+08:00
draft: false
tags: ["csharp","dotnet core"]
slug: "dotnet-core-console-di"
---
## 在 .NET Core console 上使用 Dependency Injection - DI

開始撰寫 ASP.NET Core 後，對於整個開發流程雖然不至於陌生卻也一直覺得掌握度不足，尤其在習慣 ASP.NET Core 註冊及啟動流程後，突然要寫 .NET Core console application 時卻一直不順手，似乎總少些了什麼。主要原因就是 ASP.NET Core 專案範本預設已經使用 DI，而 .NET Core console 並沒有，但現在很多套件、範例都是透過 DI 來註冊使用，為了一致的開發體驗就趁著這個機會嘗試看看如何在 .NET Core console 使用與 ASP.NET Core 相同的 DI

官方文件沒有完整的介紹，看了些文件，順手紀錄一下

## A. 簡易 DI

1. 安裝套件 - `Microsoft.Extensions.DependencyInjection`

    - Package Manager Console

        ```bash
        Install-Package Microsoft.Extensions.DependencyInjection
        ```

    - .NET CLI

        ```bash
        dotnet add package Microsoft.Extensions.DependencyInjection
        ```

2. 程式碼

    ```cs
    class Program
    {
        static void Main(string[] args)
        {
            var servicesProvider= 
                //新增 service connection
                new ServiceCollection()
                //設定每次 request 都會建立 Runner(YowkoRunner)
                .AddTransient<IRunner, Runner>()
                //建立 service provider
                .BuildServiceProvider();

            //從 service provider 中取得上面設定建立的 Runner(YowkoRunner) 服務
            var runner = servicesProvider.GetRequiredService<IRunner>();

            //透過取得的服務來執行某個動作
            runner.DoAction("Yowko");

            Console.ReadLine();
        }
    }

    public interface IRunner
    {
        void DoAction(string name);
    }
    public class Runner : IRunner
    {
        public void DoAction(string name)
        {
            Console.WriteLine($"Doing hard work! {name}");
        }
    }
    public class YowkoRunner : IRunner
    {
        public void DoAction(string name)
        {
            Console.WriteLine($"YowkoRunner : Doing hard work! {name}");
        }
    }
    ```

## B. (可能是)未來主要 DI

今天想要透過 console 寫個小工具時發現沒有用 DI 連要寫個 log 用起來都覺得怪，這也讓我聯想到 ASP.NET Core 都可以透過 Program 的 Main 來設定並啟動 host，反而 console project 沒有走相同流程？！

查了文件 [Host in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/index?WT.mc_id=DOP-MVP-5002594) 有提到 host 有兩種 API 負責應用程式啟動及 lifetime 管理

- `Web Host`

    > 主要用來 host web application

- `Generic Host`

    > 目前版本主要用來 host 非 web application (e.g. 背景執行的工作)，之後預計發展成可以同時 host 任何類型的應用程式，包含 web，也預計徹底取代 Web Host

    ![1generichost](https://user-images.githubusercontent.com/3851540/49169687-4f2ef200-f375-11e8-9a90-1c74dc70b09b.png)

1. 安裝套件
    - `Microsoft.Extensions.Hosting`
        - Package Manager Console

            ```bash
            Install-Package Microsoft.Extensions.Hosting
            ```

        - .NET CLI

            ```bash
            dotnet add package Microsoft.Extensions.Hosting
            ```  

    - `Microsoft.Extensions.DependencyInjection`
        - Package Manager Console

            ```bash
            Install-Package Microsoft.Extensions.DependencyInjection
            ```

        - .NET CLI

            ```bash
            dotnet add package Microsoft.Extensions.DependencyInjection
            ```

    - 相依套件(理論上會自動安裝，曾經遇過需手動安裝)
        - Microsoft.Extensions.Configuration
        - Microsoft.Extensions.DependencyInjection
        - Microsoft.Extensions.FileProviders.Physical
        - Microsoft.Extensions.Hosting.Abstractions
        - Microsoft.Extensions.Logging

        ![2dependency](https://user-images.githubusercontent.com/3851540/49169688-4fc78880-f375-11e8-9bcf-7724db826571.png)

2. 程式碼

    ```cs
    class Program
    {
        static void Main(string[] args)
        {
            var builder =
                //新建 HostBuilder
                new HostBuilder()
                //設定所需 service
                .ConfigureServices((hostContext, services) =>
                {
                    //設定每次 request 都會建立 Runner
                    services.AddTransient<IRunner, Runner>();
                });

            
            var runner =
                //使用上面建立的 host builder 來初始化 host
                builder.Build()
                //從 service provider 中取得上面建立的 Runner 服務
                .Services.GetRequiredService<IRunner>();

            //透過取得的服務來執行動作
            runner.DoAction("Yowko");

            Console.ReadLine();
        }
    }

    public interface IRunner
    {
        void DoAction(string name);
    }
    public class Runner : IRunner
    {
        public void DoAction(string name)
        {
            Console.WriteLine($"Doing hard work! {name}");
        }
    }
    public class YowkoRunner : IRunner
    {
        public void DoAction(string name)
        {
            Console.WriteLine($"YowkoRunner : Doing hard work! {name}");
        }
    }
    ```

## 心得

從 .NET Core 的使用上明顯可以看得出 .NET Core 仍在快速發展中，常常一陣子沒有留意就又多了一些內容出來，也造成文件的產生速度有些落後，但相較於其他 open source 工具還是好上許多，只能說以前胃口被養大了

回到這次紀錄的內容，程式碼數量上並沒有太多差異，不過整個 host 啟動與初始化流程卻有很大的不同，但說實話對於 Generic Host 的用法還是有些疑慮，主要是官方都說還在開發階段，依過去 MicroSoft 的習慣，大幅改動是非常有可能的XD 現在冒然使用難免有些風險呀

## 參考資料

1. [Host in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/index?WT.mc_id=DOP-MVP-5002594)
2. [Using IHost .net core console applications](https://garywoodfine.com/ihost-net-core-console-applications/)
3. [Dependency injection in .NET Core Console Application](https://korytiak.com/Home/BlogPost?PostName=Dependency%20injection%20in%20.NET%20Core%20Console%20Application)
4. [NET Core Console App Dependency Injection and User Secrets](https://long2know.com/2018/02/net-core-console-app-dependency-injection-and-user-secrets/)
5. [Getting started with .NET Core 2 Console application](https://github.com/NLog/NLog.Extensions.Logging/wiki/Getting-started-with-.NET-Core-2---Console-application)
