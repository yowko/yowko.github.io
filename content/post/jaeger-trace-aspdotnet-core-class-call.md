---
title: "使用 Jaeger 追蹤 ASP.NET Core 中的 class 呼叫"
date: 2019-04-14T15:30:00+08:00
lastmod: 2021-11-02T15:30:31+08:00
draft: false
tags: ["dotnet core","ASP.NET Core","Jaeger"]
slug: "jaeger-trace-aspdotnet-core-class-call"
---
## 使用 Jaeger 追蹤 ASP.NET Core 中的 class 呼叫

之前筆記 [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/) 與 [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/) 分別紀錄到使用 Jaeger 來紀錄 gRPC call 與 ASP.NET Core Web API 的呼叫歷程內容，接著紀錄另個常見使用情境：大型系統架構中，拆分多個 layer 是很常見的，如果無法正確追蹤各 layer 間的呼叫，實際應用上會大打折扣

## 基本環境說明

1. macOS Mojave 10.14.3
2. Docker Engine - Community 18.09.2
3. jaegertracing/all-in-one 1.11.0
4. NuGet package:Jaeger 0.3.1
5. NuGet package:OpenTracing.Contrib.NetCore 0.5.0
6. 接續 [yowko/Jaeger-ASP.NET-Core](https://github.com/yowko/Jaeger-ASP.NET-Core) 內容調整專案

    > 為了模擬多 layer 情境而調整專案

    - 加入新的 class library project
    - API Controller 加入使用 class library

## 建立 Jaeger 環境

> 透過 docker 建立 Jaeger 服務及後台

```bash
docker run --rm -d -p 6831:6831/udp -p 16686:16686 jaegertracing/all-in-one
```

## 安裝 NuGet 套件

> 建議將 ASP.NET Core Web API 上原加入的 NuGet package 移除，改依賴 class library 上的 NuGet 參考

1. OpenTracing.Contrib.NetCore

    > 目前最新版本為 `0.5.0`

    - Package Manager

        ```bash
        Install-Package OpenTracing.Contrib.NetCore
        ```

    - .NET CLI

        ```bash
        dotnet add package OpenTracing.Contrib.NetCore
        ```

2. Jaeger

    > 目前最新版本為 `0.3.1`

    - Package Manager

        ```bash
        Install-Package Jaeger
        ```

    - .NET CLI

        ```bash
        dotnet add package Jaeger
        ```

## 為 Class Library 加入 Jaeger trace

1. 使用 DI 引用 tracer

    ```cs
    private readonly ITracer _tracer;

    public ResponseHelper(ITracer tracer)
    {
        _tracer = tracer;
    }
    ```

2. 建立 span

    ```cs
    using (IScope scope = _tracer.BuildSpan(GetType().Name).StartActive(finishSpanOnDispose: true))
    {
    }
    ```

3. 加入 tag 以及 log

    ```cs
    scope.Span.SetTag("module", $"{GetType().Name}:{System.Reflection.MethodBase.GetCurrentMethod().Name}");

    scope.Span.Log(new Dictionary<string, object>
    {
        ["module"] = GetType().Name,
        ["function"] = System.Reflection.MethodBase.GetCurrentMethod().Name,
        ["eventTime"] = DateTime.Now,
        ["input"] = values,
        ["output"] = output
    });
    ```

4. 完整 ResponseHelper 內容

    ```cs
    using System;
    using System.Collections.Generic;
    using OpenTracing;

    namespace Helpers
    {
        public class ResponseHelper
        {
            private readonly ITracer _tracer;

            public ResponseHelper(ITracer tracer)
            {
                _tracer = tracer;
            }

            public string GetResponse(string[] values)
            {
                using (IScope scope = _tracer.BuildSpan(GetType().Name).StartActive(finishSpanOnDispose: true))
                {
                    string output = $"return \"{string.Join(',', values)}\" @ {DateTimeOffset.UtcNow}";
                    scope.Span.SetTag("module", $"{GetType().Name}:{System.Reflection.MethodBase.GetCurrentMethod().Name}");

                    scope.Span.Log(new Dictionary<string, object>
                    {
                        ["module"] = GetType().Name,
                        ["function"] = System.Reflection.MethodBase.GetCurrentMethod().Name,
                        ["eventTime"] = DateTime.Now,
                        ["input"] = values,
                        ["output"] = output
                    });

                    return output;
                }
            }
        }
    }
    ```

## 實際效果

![1result](https://user-images.githubusercontent.com/3851540/56089942-209d5800-5ecd-11e9-9fd8-69d080d775da.png)

## 心得

原本覺得 Jaeger 在 library 的呼叫使用上需要自己手刻 tag 與 log 內容  似乎有些麻煩，但仔細想想這個動作跟一般 log 是相同的，其實也合理，不過如此一來相同內容就需要重複手刻二次，身為一個懶惰工程師有點難接受呀，不過先求有  有機會再來找找有沒有更便捷的方法吧

完成程式碼請參考 [yowko/Jaeger-ASP.NET-Core](https://github.com/yowko/Jaeger-ASP.NET-Core)

## 參考資訊

1. [.NET Core 上使用 Jaeger 追蹤 gRPC 呼叫](/dotnet-core-jaeger-grpc/)
2. [使用 Jaeger 追蹤 ASP.NET Core 呼叫](/jaeger-trace-aspdotnet-core/)
3. [jaegertracing/jaeger-client-csharp](https://github.com/jaegertracing/jaeger-client-csharp)
