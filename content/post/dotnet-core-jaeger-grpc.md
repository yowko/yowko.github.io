---
title: ".NET Core 上使用 Jaeger 追蹤 gRPC 呼叫"
date: 2019-03-13T21:30:00+08:00
lastmod: 2019-03-13T21:30:31+08:00
draft: false
tags: ["dotnet core","C#","gRPC","Protobuf","Jaeger"]
slug: "dotnet-core-jaeger-grpc"
---
# .NET Core 上使用 Jaeger 追蹤 gRPC 呼叫
隨著系統使用人數愈來愈多，架構也跟著愈來愈複雜，各種技術為了解決既有問題或是加快反應速度不斷推陳出新，服務的架構也從單一系統變成多層式設計再轉為微服務，但不變的還是需要 trace log 來解決 bug，為了解決日益困難的 trace 問題，2010 年 Google 發表了 [Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](https://ai.google/research/pubs/pub36356) 論文，立下了分散式追蹤系統的根基，有很多系統就是基於該篇論文內容的實作，包含了 Zipkin、Jaeger ....etc

所在團隊需要一套可以 trace ASP.NET Core + gRPC 的軟體，過程中嘗試了 Zipkin、Jaeger、SkyWalking，最後選擇了 Jaeger，剛完成 .NET Core 上使用 Jaeger 追蹤 gRPC 呼叫的 POC, 立馬紀錄一下，不然我相信我一定記不了多久的XD

## 基本環境說明
1. macOS Mojave 10.14.3
2. Docker Engine - Community 18.09.2
3. Grpc 1.19.0
4. Grpc.Tools 1.19.0
5. Google.Protobuf 3.7.0
6. Google.Protobuf.Tools 3.7.0
7. OpenTracing.Contrib.Grpc 0.1.0
8. Jaeger 0.2.2
9. Microsoft.Extensions.Logging.Console 2.2.0
10. ProtoBuf 定義

    - message

        ```
        syntax = "proto3";

        package gRPC.Message;
        option csharp_namespace = "gRPC.Message";
        import "google/protobuf/timestamp.proto";

        message HelloRequest{
            string Name=1;
            google.protobuf.Timestamp SendDate=2;
        }

        message GoodByeRequest{
            string Name=1;
        }

        message Response{
            bool IsSuccess=1;
            string ResponseMsg=2;
        }
        ```
    - service

        ```
        syntax = "proto3";

        package gRPC.Message; 
        option csharp_namespace = "gRPC.Message";
        import "message.proto";

        service gRPCService {
        rpc SayHello(HelloRequest) returns (Response);
        rpc SayGoodbye(GoodByeRequest) returns (Response);
        }
        ```

## 建立 Jaeger 環境

> 透過 docker 建立 Jaeger 服務及後台

```bash
docker run --rm -d -p 6831:6831/udp -p 16686:16686 jaegertracing/all-in-one
```

## 建立 gRPC.Message 共用層

1. 安裝 NuGet 套件
    - Grpc

        - Package Manager
        
            ```
            Install-Package Grpc
            ```
        - .NET CLI

            ```
            dotnet add package Grpc
            ```
    - Grpc.Tools
        - Package Manager
        
            ```
            Install-Package Grpc.Tools
            ```
        - .NET CLI

            ```
            dotnet add package Grpc.Tools
            ```
    - Google.Protobuf
        - Package Manager
        
            ```
            Install-Package Google.Protobuf
            ```
        - .NET CLI

            ```
            dotnet add package Google.Protobuf
            ``` 
    - OpenTracing.Contrib.Grpc
        - Package Manager
        
            ```
            Install-Package OpenTracing.Contrib.Grpc
            ```
        - .NET CLI

            ```
            dotnet add package OpenTracing.Contrib.Grpc
            ``` 
    - Jaeger
        - Package Manager
        
            ```
            Install-Package Jaeger
            ```
        - .NET CLI

            ```
            dotnet add package Jaeger
            ``` 
    - Microsoft.Extensions.Logging.Console
        - Package Manager
        
            ```
            Install-Package Microsoft.Extensions.Logging.Console
            ```
        - .NET CLI

            ```
            dotnet add package Microsoft.Extensions.Logging.Console
            ``` 

2. 將 protobuf 轉為 C# class

    ```
    /Users/`whoami`/.nuget/packages/grpc.tools/1.19.0/tools/macosx_x64/protoc -I /Users/`whoami`/.nuget/packages/google.protobuf.tools/3.7.0/tools/ -I ./proto/ --csharp_out src/gRPC.Message --grpc_out src/gRPC.Message ./proto/*.proto --plugin=protoc-gen-grpc=/Users/`whoami`/.nuget/packages/grpc.tools/1.19.0/tools/macosx_x64/grpc_csharp_plugin
    ```
3. 加入 TracingHelper.cs

    ```cs
    using Microsoft.Extensions.Logging;
    using Jaeger;
    using Jaeger.Samplers;

    namespace gRPC.Message
    {
        public static class TracingHelper
        {
            public static Tracer InitTracer(string serviceName, ILoggerFactory loggerFactory)
            {
                Configuration.SamplerConfiguration samplerConfiguration = new Configuration.SamplerConfiguration(loggerFactory)
                    .WithType(ConstSampler.Type)
                    .WithParam(1);

                Configuration.ReporterConfiguration reporterConfiguration = new Configuration.ReporterConfiguration(loggerFactory)
                    .WithLogSpans(true);

                return (Tracer)new Configuration(serviceName, loggerFactory)
                    .WithSampler(samplerConfiguration)
                    .WithReporter(reporterConfiguration)
                    .GetTracer();
            }
        }
    }
    ```

## 建立 gRPC.Server 
1. 將 gRPC.Message 加入參考
2. 加入 gRPCServiceImpl.cs 並繼承 gRPCService.gRPCServiceBase

    ```cs
    using System.Reflection.Metadata.Ecma335;
    using System.Threading.Tasks;
    using gRPC.Message;
    using Grpc.Core;

    namespace gRPC.Server
    {
        public class gRPCServiceImpl : gRPCService.gRPCServiceBase
        {
            public override Task<Response> SayHello(HelloRequest request, ServerCallContext context)
            {
                return new Task<Response>(() =>
                {
                    return new Response
                    {
                        IsSuccess = true,
                        ResponseMsg = $"Hi {request.Name} @ {request.SendDate.ToDateTime()} !!!"
                    };
                });
            }

            public override Task<Response> SayGoodbye(GoodByeRequest request, ServerCallContext context)
            {
                return new Task<Response>(() =>
                {
                    return new Response
                    {
                        IsSuccess = true,
                        ResponseMsg = $"Bye,{request.Name}"
                    };
                });
            }
        }
    }
    ```
3. 建立 gRPC interceptor 並啟動 gRPC server

    ```cs
    using System;
    using System.Threading.Tasks;
    using gRPC.Message;
    using Grpc.Core;
    using Grpc.Core.Interceptors;
    using Jaeger;
    using Microsoft.Extensions.Logging;
    using OpenTracing.Contrib.Grpc.Interceptors;

    namespace gRPC.Server
    {
        public class Program
        {
            private const string Server = "localhost";
            const int Port = 50051;


            public static async Task Main(string[] args)
            {
                ILoggerFactory loggerFactory = new LoggerFactory().AddConsole();
                var serviceName = "server";
                Tracer tracer = TracingHelper.InitTracer(serviceName, loggerFactory);
                ServerTracingInterceptor tracingInterceptor = new ServerTracingInterceptor(tracer);
                
                
                Grpc.Core.Server server = new Grpc.Core.Server
                {
                    Services = {gRPCService.BindService(new gRPCServiceImpl()).Intercept(tracingInterceptor)},
                    Ports = {new ServerPort(Server, Port, ServerCredentials.Insecure)}
                };
                server.Start();

                Console.WriteLine($"Greeter server listening on server {server} and port {Port}");
                Console.WriteLine("Press any key to stop the server...");
                Console.ReadKey();

                await server.ShutdownAsync();
            }
        }
    }
    ```

## 建立 gRPC.Client
1. 將 gRPC.Message 加入參考
2. 建立 gRPC interceptor 並啟動 gRPC server

    ```cs
    using System;
    using System.Threading.Tasks;
    using gRPC.Message;
    using Google.Protobuf.WellKnownTypes;
    using Grpc.Core;
    using Grpc.Core.Interceptors;
    using Jaeger;
    using Microsoft.Extensions.Logging;
    using OpenTracing.Contrib.Grpc.Interceptors;

    namespace gRPC.Client
    {
        public class Program
        {
            const string Server = "localhost";
            const int Port = 50051;

            public static async Task Main(string[] args)
            {
                var loggerFactory =new LoggerFactory().AddConsole();
                var serviceName = "client";
                Tracer tracer = TracingHelper.InitTracer(serviceName, loggerFactory);
                ClientTracingInterceptor tracingInterceptor = new ClientTracingInterceptor(tracer);


                Channel channel = new Channel($"{Server}:{Port}", ChannelCredentials.Insecure);

                var client = new gRPCService.gRPCServiceClient(channel.Intercept(tracingInterceptor));
                string user = "yowko";

                var reply = client.SayHello(new HelloRequest {Name = user,SendDate = DateTime.UtcNow.ToTimestamp()});
                Console.WriteLine("Greeting: " + reply.ResponseMsg);


                var response = client.SayGoodbye(new GoodByeRequest() {Name = user});
                Console.WriteLine("Response: " + response.ResponseMsg);

                await channel.ShutdownAsync();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }
        }
    }
    ```

## 實際使用
- 使用 http://localhost:16686 開啟 Jaeger UI

    ![1jaegerui](https://user-images.githubusercontent.com/3851540/54298274-68da0980-45f3-11e9-84d7-193c50ce1837.png)
- 依 `Service` 及 `Operation` 來搜尋

    ![2findtrace](https://user-images.githubusercontent.com/3851540/54298275-68da0980-45f3-11e9-997b-a3dd0b6d7ccb.png)
- 搜尋結果

    ![3traces](https://user-images.githubusercontent.com/3851540/54298276-6972a000-45f3-11e9-888d-ff08120f6665.png)
- trace 細節

    > 包含單一 service 內容及 service 使用順序

    ![4tracedetail](https://user-images.githubusercontent.com/3851540/54298279-6972a000-45f3-11e9-9fcb-c1bf37d317ec.png)


## 心得
前面提到嘗試了 Zipkin、Jaeger、SkyWalking，其中 Zipkin 因為在 interceptor 的處理相對複雜而出局;SkyWalking 使用的組件較多、問題也較多下決定放棄，以使用的難易度來看，Jaeger 相較 Zipkin 與 SkyWalking 下，至少輕了二個量級：架構容易、使用方式也簡單

完成程式碼內容請參考 [yowko/dotnetCore-gRPC-Jaeger](https://github.com/yowko/dotnetCore-gRPC-Jaeger.git)

# 參考資訊
1. [OpenTracing gRPC Tutorial - C#](https://github.com/opentracing-contrib/csharp-grpc/blob/master/getting_started/README.md)
2. [yowko/dotnetCore-gRPC-Jaeger](https://github.com/yowko/dotnetCore-gRPC-Jaeger.git)