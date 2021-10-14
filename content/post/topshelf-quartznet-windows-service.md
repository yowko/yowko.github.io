---
title: "使用 Topshelf 搭配 Quartz.Net 撰寫 Windows Service 排程執行工作"
date: 2018-04-12T21:00:00+08:00
lastmod: 2021-10-14T21:00:39+08:00
draft: false
tags: ["套件","C#","Windows Service"]
slug: "topshelf-quartznet-windows-service"
aliases:
    - /2018/04/topshelf-quartznet-windows-service.html
---
## 使用 Topshelf 搭配 Quartz.Net 撰寫 Windows Service 排程執行工作

排程工作在許多系統中都是必備組件，常用來處理非立即性作業(e.g.：每日交易結清，發送電子報...etc)，如果沒有特別要求或限制下做法非常多：像是 host 在 web application 下的 web background runner : Quartz.Net 與 hangfire 或是 console +  scheduled tasks ，另外就是今天要紀錄的 Windows Service + timer 功能，都可以達到排程執行目的

雖然今天要紀錄的內容目標是 Windows Service + timer，卻也有些不同，會透過 console 應用程式與 Quartz.Net 來實作，並使用 Topshelf 來安裝，其中 Topshelf 是套用來簡便建立 Windows Service 的 framework，可以直接撰寫 console 應用程式讓整個開發及偵錯更為容易，完成功能開發後再將 console 應用程式安裝為 Windows Service 而 Quartz.NET 則是排程 framework，移植自 java 上的 Quartz. 立馬來看看如何實作吧

## 實際使用方式

1. 建立 console 專案
2. 安裝 NuGet 套件 `Topshelf` 及 `Topshelf.Quartz`

    ```cs
    install-package Topshelf  
    install-package Topshelf.Quartz
    ```

3. 加入 Service

    > 用來處理 Windows Service 啟動及停止行為

    ```cs
    using System;

    namespace TestTopshelf
    {
        public class WinService
        {
            public void OnStart()
            {
                Console.WriteLine("Service Start.");
            }
    
            public void OnStop()
            {
                Console.WriteLine("Service Stop.");
            }
        }
    }
    ```

4. 加入實際需要排程執行的動作

    > 需繼承 `IJob` 並實作 `Execute`

    ```cs
    using Quartz;
    using System;
    
    namespace TestTopshelf
    {
        public class Job : IJob
        {
            public void Execute(IJobExecutionContext context)
            {
                Console.WriteLine($"[{DateTime.Now}] Execute Job!");
            }
        }
    }

    ```

5. 設定 Topshelf 及 Quartz

    ```cs
    using System;
    using Quartz;
    using Topshelf;
    using Topshelf.Quartz;
    
    namespace TestTopshelf
    {
        class Program
        {
            static void Main(string[] args)
            {
                // 執行 Windows Service 的 host 相關設定
                HostFactory.Run(x =>
                {
                    // 註冊 WinService 至 Topshelf 
                    x.Service<WinService>(s =>
                    {
                        // 註冊 WinService 初始化方法至 Topshelf 
                        s.ConstructUsing(() => new WinService());
                        // 註冊如何啟動
                        s.WhenStarted(service => service.OnStart());
                        // 註冊如何停止
                        s.WhenStopped(service => service.OnStop());
    
                        //設定 Quartz 如何執行
                        s.ScheduleQuartzJob(q =>
                            q.WithJob(() =>
                                JobBuilder.Create<Job>().Build())//建立 Job
                                .AddTrigger(() => TriggerBuilder.Create()//設定 trigger
                                    .WithSimpleSchedule(b => b
                                        .WithIntervalInSeconds(10)//10秒執行一次
                                        .RepeatForever())
                                    .Build()));
                    });
    
                    
                    x.RunAsLocalSystem()// 以 `local system` 執行
                        .StartAutomatically()//開機自動啟動
                        .EnableServiceRecovery(rc => rc.RestartService(1));//直到錯誤計數重置的天數
    
                    x.SetServiceName("WinService");//設定 service 名稱
                    x.SetDisplayName("WinService");//設定 service 顯示名稱
                    x.SetDescription("Test WinService with Topshelf and Quartz");//設定 service 描述
                });
            }
        }
    }

    ```

6. 執行
    - 直接執行 console

        >![1execute](https://user-images.githubusercontent.com/3851540/38634250-917ed094-3df4-11e8-8b80-592bbf2e72b2.png)

    - 註冊為 Windows Service

        >需以管理者權限執行

        - 指令

            ```cmd
            {console application}.exe install
            ```

        - 範例

            ```cmd
            TestTopshelf.exe install
            ```

            >![2install](https://user-images.githubusercontent.com/3851540/38634251-91b9ddec-3df4-11e8-8990-743513a41131.png)
            >![3install2](https://user-images.githubusercontent.com/3851540/38634252-91f1d5c6-3df4-11e8-99fa-53d43c41519e.png)

## 心得

一開始提到可以達成排程工作的做法很多元，不同做法間各有優缺點也有不同的適用情境，大致上我自己的挑選方式是：

1. web background runner

    >沒有專屬的 job server 時

    - 好處是跟網站在一起有問題會立馬發現
    - 缺點也是跟網站在一起會資源互相影響

2. windows service

    >有專屬 job server 且需持續或固定時間執行

    - 好處是針對頻繁或耗時作業有控制權 (遇到上次作業未結束可以略過不跑)
    - 缺點較為耗資源，修改需要重新安裝、不好 debug (最後兩個缺點使用 topshelf 可解決)

3. scheduled task

    >有專屬 job server 且需要可以彈性設定執行時間或是頻率

    - 好處是資源耗用較低，執行頻率可彈性設定
    - 缺點是上個作業尚未完成，可能會重複執行

依我自己的經驗是各個方式都出現過問題，還是需要有其他備援或是監控機制比較保險，但回歸到今天的內容：Topshelf 與 Quartz.NET 是非常優秀的套件，讓解決方案不且更為豐富方便也相當容易上手，值得學習

## 參考資訊

1. [Scheduled jobs made easy – Topshelf and Quartz.NET](http://www.mpustelak.com/2017/01/scheduled-jobs-made-easy-topshelf-quartz-net/)
2. [Configuring Topshelf » Show me the code!](https://topshelf.readthedocs.io/en/latest/configuration/quickstart.html)
3. [Quartz.NET Quick Start Guide](https://www.quartz-scheduler.net/documentation/quartz-3.x/quick-start.html)
4. [Topshelf Command-Line Reference](http://docs.topshelf-project.com/en/latest/overview/commandline.html)
5. [//TODONT: Use a Windows Service just to run a scheduled process](https://weblogs.asp.net/jongalloway/428303)
6. [scheduled task or windows service](https://stackoverflow.com/questions/1460580/scheduled-task-or-windows-service)
7. [windows service vs scheduled task](https://stackoverflow.com/questions/390307/windows-service-vs-scheduled-task)
