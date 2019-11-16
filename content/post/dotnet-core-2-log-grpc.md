---
title: "C# (.NET Core 2) Log 與 Trace gRPC"
date: 2019-11-12T21:30:00+08:00
lastmod: 2019-11-12T21:30:31+08:00
draft: false
tags: ["csharp","gRPC"]
slug: "dotnet-core-2-log-grpc"
---

## C# (.NET Core 2) Log 與 Trace gRPC

gRPC 在 .NET Core 3 被官方宣告重點發展項目之一，而身為追求系統更快更好又愛嚐鮮的工程師團隊的一員，早在一年前的 .NET Core 2 專案中就用上了 gRPC，雖然 .NET Core 3 已正式發表，不過現在開發主力還是放在專案功能上，暫時還無暇升級，也因為還需要在 .NET Core 2 使用 gRPC，就得克服一些使用不便的難題，今天的 log 與 trace 就是其中一個

在 gRPC 的 C# repository 中有個 issue [Getting gRPC traces is hard in C#](https://github.com/grpc/grpc/issues/10574) 反應的也是同件事，看著這個 issue [Getting gRPC traces is hard in C#](https://github.com/grpc/grpc/issues/10574) 已經被 closed，也有對應的處理方式：

- 設定環境變數 `GRPC_TRACE`
- 設定環境變數 `GRPC_VERBOSITY`
- 使用 `GrpcEnvironment.SetLogger(new ConsoleLogger());` 來輸出內容

參考 [gRPC compression in C#](https://stackoverflow.com/questions/49031763/grpc-compression-in-c-sharp) 的做法，沒能順利輸出相關資訊，紀錄一下個人做法

根據 C 語言的 gRPC 實作可以參考官方的除錯建議：[Troubleshooting gRPC](https://github.com/grpc/grpc/blob/master/TROUBLESHOOTING.md) - 可以透過設定 `環境變數` 來取得更多的 gRPC 訊息以進行偵錯

## 基本環境說明

1. macOS Majave 10.14.16
2. .NET Core SDK 2.2.301
3. JetBrains Rider 2019.2.2
4. ASP.NET Core MVC 2.2 預設專案範本

    > 程式碼部份可參考 [C# 搭配 gRPC 中使用 stream RPC](https://blog.yowko.com/csharp-grpc-stream/)

5. NuGet Package

    - Google.Protobuf 3.8.0
    - Grpc 2.25.0
    - Grpc.Tools 2.25.0

## 設定說明

1. 使用 `GrpcEnvironment.SetLogger(new ConsoleLogger());` 來輸出內容

    > 程式碼中加入 `GrpcEnvironment.SetLogger(new ConsoleLogger());` 立馬就可以發現多了 gRPC 的 log

    ```txt
    D1116 18:09:56.061385 Grpc.Core.Internal.UnmanagedLibrary Attempting to load native library "/Users/yowko.tsai/.nuget/packages/grpc.core/2.25.0/lib/netstandard2.0/../../runtimes/osx/native/libgrpc_csharp_ext.x64.dylib"
    D1116 18:09:56.105759 Grpc.Core.Internal.NativeExtension gRPC native library loaded successfully.
    ```

    ![1load](https://user-images.githubusercontent.com/3851540/68995144-435d7800-08c5-11ea-98fc-6c9262b62b6d.png)

2. 設定環境變數 `GRPC_VERBOSITY` 為 `DEBUG`

    > 除了載入 native library 的訊息外，會多輸出其他 debug log，預設可用的設定值如下

    - DEBUG : 會紀錄所有 gRPC 訊息
    - INFO : 紀錄 `INFO` 與 `ERROR` 訊息
    - ERROR : 僅紀錄 `ERROR` 訊息

    ![2debuglog](https://user-images.githubusercontent.com/3851540/68995145-435d7800-08c5-11ea-82ae-1844e44aa7a3.png)

3. 設定環境變數 `GRPC_TRACE`

    > 可以設定不同的 tracer 來輸出需要的 log，可以使用的 trace 如下 (多數 tracer 我都沒用過，很多專有名詞也不懂，就不翻譯免得搞錯)，內容截取至 [gRPC environment variables](https://github.com/grpc/grpc/blob/master/doc/environment_variables.md)

    - api : traces api calls to the C core
    - bdp_estimator - traces behavior of bdp estimation logic
    - call_error - traces the possible errors contributing to final call status
    - cares_resolver - traces operations of the c-ares based DNS resolver
    - cares_address_sorting - traces operations of the c-ares based DNS resolver's resolved address sorter
    - channel - traces operations on the C core channel stack
    - client_channel_call - traces client channel call batch activity
    - client_channel_routing - traces client channel call routing, including resolver and load balancing policy interaction
    - compression - traces compression operations
    - connectivity_state - traces connectivity state changes to channels
    - cronet - traces state in the cronet transport engine
    - executor - traces grpc's internal thread pool ('the executor')
    - glb - traces the grpclb load balancer
    - handshaker - traces handshaking state
    - health_check_client - traces health checking client code
    - http - traces state in the http2 transport engine
    - http2_stream_state - traces all http2 stream state mutations.
    - http1 - traces HTTP/1.x operations performed by gRPC
    - inproc - traces the in-process transport
    - flowctl - traces http2 flow control
    - op_failure - traces error information when failure is pushed onto a completion queue
    - pick_first - traces the pick first load balancing policy
    - plugin_credentials - traces plugin credentials
    - pollable_refcount - traces reference counting of 'pollable' objects (only in DEBUG)
    - resource_quota - trace resource quota objects internals
    - round_robin - traces the round_robin load balancing policy
    - queue_pluck
    - server_channel - lightweight trace of significant server channel events
    - secure_endpoint - traces bytes flowing through encrypted channels
    - subchannel - traces the connectivity state of subchannel
    - timer - timers (alarms) in the grpc internals
    - timer_check - more detailed trace of timer logic in grpc internals
    - transport_security - traces metadata about secure channel establishment
    - tcp - traces bytes in and out of a channel
    - tsi - traces tsi transport security

    > - 使用 `all` 會啟用所有 tracer
    > - 在特定的 tracer 前面加上 `-` 前綴可以停用特定 tracer
    > - `list_tracers` 會列出當前環境可以使用的 tracer

## 實際使用

參考 [gRPC compression in C#](https://stackoverflow.com/questions/49031763/grpc-compression-in-c-sharp) 使用 (個人在 mac 搭 Rider 無法成功輸出設定的 log)

```cs
Environment.SetEnvironmentVariable("GRPC_TRACE", "all");
Environment.SetEnvironmentVariable("GRPC_VERBOSITY", "DEBUG");
GrpcEnvironment.SetLogger(new ConsoleLogger());
```

一直無法成功輸出對應的 log，主要是 `GRPC_TRACE` 沒生效，`GRPC_VERBOSITY` 到是一切正常，後來無意間嘗試由執行當下設定環境變數的做法就成功了

- dotnet cli

    ```bash
    GRPC_TRACE=compression,list_tracers GRPC_VERBOSITY=DEBUG ASPNETCORE_ENVIRONMENT=Development dotnet ./GRpc.Server/bin/Debug/netcoreapp2.2/GRpc.Server.dll
    ```

- 或是由 Rider 的 config 中設定

    ![3ridersetting](https://user-images.githubusercontent.com/3851540/68995146-435d7800-08c5-11ea-965d-da4918189d65.png)

## 心得

[Getting gRPC traces is hard in C#](https://github.com/grpc/grpc/issues/10574) 很簡單地帶過用法，試了 [gRPC compression in C#](https://stackoverflow.com/questions/49031763/grpc-compression-in-c-sharp) 的做法卻無法正確輸出 log，之前放棄一次，但為了除錯再嘗試，終於在反覆測試後才找到原來是 `GRPC_TRACE` 在環境變數的設定上有問題，花超多時間的，一度以為可能是 .NET 2.2 搭配 gRPC 的原生限制

雖然最後問題解決了，但我心裡閃過幾次念頭：會不會是 mac 或是 Rider 的問題，不過個人猜測這個問題應該只有同個 team 的同事會遇到，就沒有另外找機器跟時間進行測試

## 參考資訊

1. [Getting gRPC traces is hard in C#](https://github.com/grpc/grpc/issues/10574)
2. [gRPC compression in C#](https://stackoverflow.com/questions/49031763/grpc-compression-in-c-sharp)
3. [Troubleshooting gRPC](https://github.com/grpc/grpc/blob/master/TROUBLESHOOTING.md)
4. [gRPC environment variables](https://github.com/grpc/grpc/blob/master/doc/environment_variables.md)
