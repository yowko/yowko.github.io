---
title: "ASP.NET Core 的 gRPC-Web 功能"
date: 2023-05-05T00:30:00+08:00
lastmod: 2023-05-05T00:30:31+08:00
draft: false
tags: ["csharp","grpc","dotnet","aspdotnetcore"]
slug: "aspdotnetcore-grpc-web"
---

## ASP.NET Core 的 gRPC-Web 功能

之前筆記 [gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json) 紀錄到如何使用 Transcoding 讓 gRPC service 同時提供 web rest api 的功能，過程中在 Microsoft 官方文件 [gRPC JSON transcoding in ASP.NET Core gRPC apps](https://learn.microsoft.com/en-us/aspnet/core/grpc/json-transcoding?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594) 看到  Microsoft 官方將 gRPC JSON transcoding 與 gRPC-Web 做比較，所以趁著這個機會一併了解一下相關用法

因為瀏覽器主要使用 HTTP1.1 而 gRPC 則是以 HTTP2 為主，針對這個問題 gRPC 團隊提出的 solution 是在 browser client 與 gRPC service 中間加上一個 proxy (gRPC 團隊使用 Envoy) 用來做 protocol 的轉換，並在 browser 上使用 gRPC-Web (用來連線 proxy 而開發的 JavaScript client library) 來呼叫 gRPC service

![grpc-web](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*mAkZWyRD9gKyBEOaqEFm-A.png)

> 圖片來源 [Envoy and gRPC-Web: a fresh new alternative to REST](https://blog.envoyproxy.io/envoy-and-grpc-web-a-fresh-new-alternative-to-rest-6504ce7eb880)

ASP.NET Core gRPC-Web 則是不需要額外建置 Envoy 這個 proxy，改由 Grpc.AspNetCore.Web middleware 取代，整體使用上會更加簡潔

## 基本環境說明

1. macOS Ventura 13.2
2. .NET SDK 7.0.203
3. JetBrains Rider 2023.1

    > 使用 gRPC service 預設專案範本

4. NuGet packages
    - Service
        - Grpc.AspNetCore.Web 2.52.0
    - Client
        - Grpc.Net.Client 2.52.0
        - Grpc.Net.Client.Web 2.52.0
        - Grpc.Tools 2.54.0
        - Google.Protobuf 3.22.4

## 設定方式

1. 新增 NuGet package : Grpc.AspNetCore.Web 2.52.0
2. 註冊 gRPC-Web (修改 `Program.cs`)

    - 2-1. 逐一啟用
        - 前

            ```cs
            using GrpcService1.Services;

            var builder = WebApplication.CreateBuilder(args);
            
            // Additional configuration is required to successfully run gRPC on macOS.
            // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go.        microsoft.com/fwlink/?linkid=2099682
            
            // Add services to the container.
            builder.Services.AddGrpc();
            
            var app = builder.Build();
            
            // Configure the HTTP request pipeline.
            app.MapGrpcService<GreeterService>();
            app.MapGet("/",
                () =>
                    "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
            
            app.Run();
            ```

        - 後

            ```cs
            using GrpcService1.Services;

            var builder = WebApplication.CreateBuilder(args);
            
            // Additional configuration is required to successfully run gRPC on macOS.
            // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go. microsoft.com/fwlink/?linkid=2099682
            
            // Add services to the container.
            builder.Services.AddGrpc();
            
            var app = builder.Build();
            
            //新增下列這行：使用 gRPC-Web middleware
            app.UseGrpcWeb();

            // 新增 `.EnableGrpcWeb();` 在 `GreeterService` 啟用 gRPC-Web
            app.MapGrpcService<GreeterService>().EnableGrpcWeb();
            app.MapGet("/",
                () =>
                    "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
            
            app.Run();
            ```

    - 2-2. 預設全部啟用

        - 前

            ```cs
            using GrpcService1.Services;

            var builder = WebApplication.CreateBuilder(args);
            
            // Additional configuration is required to successfully run gRPC on macOS.
            // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go.        microsoft.com/fwlink/?linkid=2099682
            
            // Add services to the container.
            builder.Services.AddGrpc();
            
            var app = builder.Build();
            
            // Configure the HTTP request pipeline.
            app.MapGrpcService<GreeterService>();
            app.MapGet("/",
                () =>
                    "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
            
            app.Run();
            ```

        - 後

            ```cs
            using GrpcService1.Services;

            var builder = WebApplication.CreateBuilder(args);
            
            // Additional configuration is required to successfully run gRPC on macOS.
            // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go.        microsoft.com/fwlink/?linkid=2099682
            
            // Add services to the container.
            builder.Services.AddGrpc();
            
            var app = builder.Build();

            //新增下列這行：使用 gRPC-Web middleware 並預設全部啟用 gRPC-Web
            app.UseGrpcWeb(new GrpcWebOptions { DefaultEnabled = true });
            
            // Configure the HTTP request pipeline.
            app.MapGrpcService<GreeterService>();
            app.MapGet("/",
                () =>
                    "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
            
            app.Run();
            ```

## client 呼叫方式

```cs
using Greet;
using Grpc.Net.Client;
using Grpc.Net.Client.Web;

var channel = GrpcChannel.ForAddress("http://localhost:5226", new GrpcChannelOptions
{
    //使用 gRPC-Web (Content-Type 會是 application/grpc-web 或 application/grpc-web-text)
    HttpHandler = new GrpcWebHandler(new HttpClientHandler())
});


var client = new  Greeter.GreeterClient(channel);


Console.WriteLine($"{ (await client.SayHelloAsync(new HelloRequest(){Name = "gRPCCall"})).Message}");
```

## 心得

1. protocol 本身的限制，只能支援 `Unary RPCs` 與 `Server-side Streaming RPCs`
2. 設定簡單，還不用自行設定 Envoy proxy
3. 需要使用 gRPC 方式來使用，不像 gRPC JSON transcoding 可以直接透過 rest api 方式呼叫

完整程式碼：[yowko/aspnetcore-grpc-web](https://github.com/yowko/aspnetcore-grpc-web)

## 參考資訊

1. [gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json)
2. [gRPC JSON transcoding in ASP.NET Core gRPC apps](https://learn.microsoft.com/en-us/aspnet/core/grpc/json-transcoding?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594)
3. [ASP.NET Core gRPC 應用程式中的 gRPC-Web](https://learn.microsoft.com/zh-tw/aspnet/core/grpc/grpcweb?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594)
4. [Grpc.AspNetCore.Web](https://www.nuget.org/packages/Grpc.AspNetCore.Web)
5. [gRPC Web](https://github.com/grpc/grpc-web)
6. [yowko/aspnetcore-grpc-web](https://github.com/yowko/aspnetcore-grpc-web)
7. [Envoy and gRPC-Web: a fresh new alternative to REST](https://blog.envoyproxy.io/envoy-and-grpc-web-a-fresh-new-alternative-to-rest-6504ce7eb880)
