---
title: "讓 ASP.NET Core MVC 自行停止運作"
date: 2019-05-12T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["ASP.NET Core MVC","ASP.NET Core"]
slug: "aspdotnet-core-mvc-stop-by-itself"
---
## 讓 ASP.NET Core MVC 自行停止運作

公司專案有個系統在啟動時執行 init script，接著就等待後台管理者下指令才會有後續動作，但開發階段暫時還不會有後台管理者的角色，因此希望可以透過參數設定來決定是否讓該系統持續提供服務或是執行工作完成後即可自行關閉以節省資源，紀錄一下加深印象

## 基本環境說明

1. macOS Mojave 10.14.4
2. Docker Engine - Community 18.09.2
3. .NET Core 2.2.101
4. ASP.NET Core  2.2.0
5. 預設 Web App (Model-View-Controller) 專案範本

## 如何修改

1. 在 `Start.cs` 中加入 `IApplicationLifetime` field

    ```cs
    private IApplicationLifetime _applicationLifetime;
    ```

2. 在 `Start.cs` 的 `Configue` 方法中加入 `IApplicationLifetime applicationLifetime` 參數並將參數值設定給先前新增的 field

    ```cs
    public void Configure(IApplicationBuilder app, IHostingEnvironment env,IApplicationLifetime applicationLifetime)
    {

            _applicationLifetime = applicationLifetime;
            //以下原設定內容略過....
    }
    ```

3. 設定將 application 關閉

    > 執行 `_applicationLifetime.StopApplication();`

    - 在 `Start.cs` 的 `Configue` 執行

        我自己在專案上使用啟動參數 (`stop`)來決定是否關閉

        ```cs
        var stop = Configuration.GetValue<bool>("stop");
        if (stop)
            _applicationLifetime.StopApplication();
        ```

        使用下列指令即可執行關閉

        ```bash
        dotnet {project}.dll stop=true  
        ```

    - 在 Controller 中執行

        - 加入 field 並在建構子允許 DI

            ```cs
            private IApplicationLifetime _applicationLifetime;

            public HomeController(IApplicationLifetime applicationLifetime)
            {
                _applicationLifetime = applicationLifetime;
            }
            ```

        - Action 執行 shutdown

            ```cs
            public void Shutdown()
            {
                _applicationLifetime.StopApplication();
            }
            ```

## 心得

`IApplicationLifetime` 有三個 Properties

1. ApplicationStarted

    > application 完全啟動後等待正常關閉時觸發 (Startup.Configure()方法結束)

2. ApplicationStopping

    > application 執行正常關閉時觸發，會待所有 request 處理完才執行關閉

3. ApplicationStopping

    > application 完成正常關閉後觸發

一個方法

1. StopApplication()

## 參考資訊

1. [如何以編程方式重啟 ASP.NET Core 網站](http://www.dalbll.com/Group/Topic/ASP.NET/7315)
2. [IApplicationLifetime Interface](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime?WT.mc_id=DOP-MVP-5002594)
