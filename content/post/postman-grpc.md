---
title: "使用 Postman 來發送 gRPC request"
date: 2023-04-28T00:30:00+08:00
lastmod: 2023-04-28T00:30:31+08:00
draft: false
tags: ["csharp","grpc","dotnet","aspdotnetcore","tools"]
slug: "postman-grpc"
---

## 使用 Postman 來發送 gRPC request

過去在測試 gRPC 時大部份都是依賴 [BloomRPC](https://github.com/bloomrpc/bloomrpc)，畢竟 grpcurl 還是語法上還是沒辦法像 GUI 一樣直覺，只是今年要更新 [BloomRPC](https://github.com/bloomrpc/bloomrpc) 時發現已停止維護，雖然 [BloomRPC](https://github.com/bloomrpc/bloomrpc) 有提供替代選擇 [awesome-grpc](https://github.com/grpc-ecosystem/awesome-grpc#tools)，但要重新評估工具可能耗日曠時，剛好之前查 [Microsoft 官方文件：Test gRPC services with Postman or gRPCurl in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/grpc/test-tools?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594) 看到 Postman 現在也支援呼叫 gRPC service，既然是老朋友有一定的熟悉度就來試試吧

## 基本環境說明

1. macOS Ventura 13.2
2. .NET SDK 7.0.203
3. JetBrains Rider 2023.1

    > 使用 gRPC service 預設專案範本，相關內容使用之前筆記內容範例 [再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json-post)、[再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能 (Streaming)](grpc-aspdotnetcore7-json-streaming)

4. NuGet packages
    - Microsoft.AspNetCore.Grpc.JsonTranscoding 7.0.5
    - Grpc.AspNetCore.Server.Reflection 2.52.0
5. Postman April 2023 (v10.13)

## 使用方式

要登入 Postman 後才能在 Workspace 下新增 gRPC Request

![1loginrequired](https://user-images.githubusercontent.com/3851540/235125427-6639e836-a13f-4407-b562-141eb0cc0ef0.png)

1. 使用 proto

    - import .proto file

        ![2import](https://user-images.githubusercontent.com/3851540/235125433-5b88bf0e-4949-4c9e-a52d-e687a17b0773.png)

    - import 其他 proto 的路徑

        ![3importpath](https://user-images.githubusercontent.com/3851540/235125434-bf69a60c-6af9-4aaa-998d-48b3c7347a3b.png)

        - 路徑不正確會提示

            ![4patherror](https://user-images.githubusercontent.com/3851540/235125440-b3ba90c3-1c04-49f0-9ca8-4fc187861378.png)

    - 將 proto 檔定義的 service 與 method 儲存為 API

        > 這一步只是為了在 Postman 中便於管理 proto 中定義的 service 與 method

        ![5api](https://user-images.githubusercontent.com/3851540/235125442-6592ea0e-d651-4ba2-ba7e-dd094e1fc793.png)

    - 儲存完成後就可以透過 API 來列出 service 與 method

        ![6using](https://user-images.githubusercontent.com/3851540/235125444-d203c9c7-f1fe-4816-87c5-b15ffac0de29.png)

        ![7methods](https://user-images.githubusercontent.com/3851540/235125446-dbcd67ff-cfe7-4279-baf2-04a17f2c2c4b.png)

2. 使用 reflection (我的環境無法使用)

    - 調整 gRPC Service

        - 新增 NuGet 套件：`Grpc.AspNetCore.Server.Reflection`
        - reflection 設定 (修改 Programs.cs)
            - 前

                ```cs
                using NET7GrpcService.Services;

                var builder = WebApplication.CreateBuilder(args);

                // Additional configuration is required to successfully run gRPC on macOS.
                // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go.microsoft.com/fwlink/?linkid=2099682

                // Add services to the container.
                builder.Services.AddGrpc().AddJsonTranscoding();
                
                var app = builder.Build();
                
                // Configure the HTTP request pipeline.
                app.MapGrpcService<GreeterService>();
                app.MapGrpcService<ExampleService>();
                app.MapGet("/",
                    () =>
                        "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

                app.Run();
                ```

            - 後

                ```cs
                using NET7GrpcService.Services;

                var builder = WebApplication.CreateBuilder(args);

                // Additional configuration is required to successfully run gRPC on macOS.
                // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go.microsoft.com/fwlink/?linkid=2099682

                // Add services to the container.
                builder.Services.AddGrpc().AddJsonTranscoding();
                // 加上下面這行
                builder.Services.AddGrpcReflection();
                
                var app = builder.Build();
                
                // Configure the HTTP request pipeline.
                app.MapGrpcService<GreeterService>();
                app.MapGrpcService<ExampleService>();
                // 加上下面四行
                if (app.Environment.IsDevelopment())
                {
                    app.MapGrpcReflectionService();
                }

                app.MapGet("/",
                    () =>
                        "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

                app.Run();
                ```

    - Using server reflection

        - 出現 unknown error

            ![8reflectionerror](https://user-images.githubusercontent.com/3851540/235125450-5aaf24e7-5d0d-4391-9fa2-ea66e8a2694d.png)

        - 使用 grpcurl 則是正常

            > 用法可以參考之前筆記 [在 ASP.NET Core 上啟用 gRPC Reflection](/aspdotnet-core-grpc-reflection/)

            ![9grpcurl](https://user-images.githubusercontent.com/3851540/235125452-b95e6099-96c6-413e-871a-1a5178560dc8.png)

## 心得

Postman UI 有加強過：四個 gRPC 方法都有不同圖示

![10icon](https://user-images.githubusercontent.com/3851540/235125455-4e2a6bd6-118f-4199-8fa2-a67d2e20692c.png)

雖然 Using server reflection 有 bug 不能使用，但整體使用體驗還不差

## 參考資訊

1. [BloomRPC](https://github.com/bloomrpc/bloomrpc)
2. [Microsoft 官方文件：Test gRPC services with Postman or gRPCurl in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/grpc/test-tools?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594)
3. [再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json-post)
4. [再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能 (Streaming)](grpc-aspdotnetcore7-json-streaming)
5. [在 ASP.NET Core 上啟用 gRPC Reflection](/aspdotnet-core-grpc-reflection/)
