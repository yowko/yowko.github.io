---
title: "使用 Jaeger 追蹤 ASP.NET Core 呼叫"
date: 2019-04-13T21:30:00+08:00
lastmod: 2019-04-13T21:30:31+08:00
draft: false
tags: ["dotnet core","ASP.NET Core","Jaeger"]
slug: "jaeger-trace-aspdotnet-core"
---
# 使用 Jaeger 追蹤 ASP.NET Core 呼叫

之前筆記 [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](https://blog.yowko.com/dotnet-core-jaeger-grpc/) 紀錄到如何使用 Jaeger 來追蹤 .NET Core Console project 中 gRPC 的呼叫歷程，不過想必未來應該免不了還是需要使用 REST api 的專案，趁著假日有空順手做個紀錄備忘 



## 基本環境說明
1. macOS Mojave 10.14.3
2. Docker Engine - Community 18.09.2
3. jaegertracing/all-in-one 1.11.0
4. NuGet package:Jaeger 0.3.1
5. NuGet package:OpenTracing.Contrib.NetCore 0.5.0
6. 使用 ASP.NET Core Web API 專案範本

## 建立 Jaeger 環境

> 透過 docker 建立 Jaeger 服務及後台

```bash
docker run --rm -d -p 6831:6831/udp -p 16686:16686 jaegertracing/all-in-one
```

## 安裝 NuGet 套件
1. OpenTracing.Contrib.NetCore

    > 目前最新版本為 `0.5.0`

    - Package Manager

        ```
        Install-Package OpenTracing.Contrib.NetCore
        ```

    - .NET CLI

        ```
        dotnet add package OpenTracing.Contrib.NetCore
        ```

2. Jaeger

    > 目前最新版本為 `0.3.1`

    - Package Manager
    
        ```
        Install-Package Jaeger
        ```
    
    - .NET CLI

        ```
        dotnet add package Jaeger
        ```

## 實際使用：修改 Startup.cs

> 修改 `ConfigureServices` function

1. 指定想要顯示在 Jaeger 上的名稱

    > 這邊使用專案名稱

    ```cs
    string serviceName = AppDomain.CurrentDomain.FriendlyName;
    ```

2. 建立 tracer

    > 使用上面建立的 serviceName 來建立 trace

    ```cs
    var tracer = new Tracer.Builder(serviceName)
                .WithSampler(new ConstSampler(true))
                .Build();
    ```

3. 將 OpenTracing 註冊進 ASP.NET Core

    ```cs
    services.AddOpenTracing();
    ```

4. 將上面建立的 trace 註冊至 OpenTracing 中

    ```cs
    services.AddSingleton<ITracer>(serviceProvider =>
    {
        //註冊 Jaeger tracer
        GlobalTracer.Register(tracer);

        return tracer;
    });
    ```
5. 完整 Startup.cs 內容

    ```cs
    using System;
    using Jaeger;
    using Jaeger.Samplers;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using OpenTracing;
    using OpenTracing.Util;

    namespace JaegerASP.NETCore
    {
        public class Startup
        {
            public Startup(IConfiguration configuration)
            {
                Configuration = configuration;
            }

            public IConfiguration Configuration { get; }

            // This method gets called by the runtime. Use this method to add services to the container.
            public void ConfigureServices(IServiceCollection services)
            {
                //指定顯示名稱
                string serviceName = AppDomain.CurrentDomain.FriendlyName;
                //建立 trace
                var tracer = new Tracer.Builder(serviceName)
                    .WithSampler(new ConstSampler(true))
                    .Build();

                //加入 OpenTracing
                services.AddOpenTracing();
                services.AddSingleton<ITracer>(serviceProvider =>
                {
                    //註冊 Jaeger tracer
                    GlobalTracer.Register(tracer);

                    return tracer;
                });


                services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
            }

            // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
            public void Configure(IApplicationBuilder app, IHostingEnvironment env)
            {
                if (env.IsDevelopment())
                {
                    app.UseDeveloperExceptionPage();
                }
                else
                {
                    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                    app.UseHsts();
                }

                app.UseHttpsRedirection();
                app.UseMvc();
            }
        }
    }
    ```

## 實際效果

![jaegerresult](https://user-images.githubusercontent.com/3851540/56081705-e1c2c000-5e42-11e9-91c5-25e9a0e2ad64.png)

## 心得
仔細看是不是發現我沒做什麼事XD，因為 `OpenTracing.Contrib.NetCore` 已經完成了大部份工作，我們只需要在實際使用時將 Jaeger tracer 註冊進去就行了, 其中實際效果的截圖中可以看到 `OpenTracing.Contrib.NetCore` 不僅把事情做完了，還打了 tag 跟加了不少內容在 log 中

另外我還在試著搞清楚的是：測試時只有一次 request ，但 Jaeger 卻收到三個 tracer 的資料

![liats](https://user-images.githubusercontent.com/3851540/56081706-e1c2c000-5e42-11e9-941d-ee3d1b986035.png)

完整專案請參考 [yowko/Jaeger-ASP.NET-Core](https://github.com/yowko/Jaeger-ASP.NET-Core)

# 參考資訊
1. [Jaeger](https://www.jaegertracing.io/)
2. [yowko/Jaeger-ASP.NET-Core](https://github.com/yowko/Jaeger-ASP.NET-Core)