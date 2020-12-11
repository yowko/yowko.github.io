---
title: "在 ASP.NET Core 上啟用 gRPC Reflection"
date: 2020-09-10T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC"]
slug: "aspdotnet-core-grpc-reflection"
---

## 在 ASP.NET Core 上啟用 gRPC Reflection

系統功能愈來愈多，自然而然地 proto 檔也就熟變得愈來愈龐大，如果每次想要手動測試 gRPC 功能時都要手 key proto 實在沒效率 (測試工具可以參考之前筆記 [使用grpcurl 呼叫gRPC Service](/grpcurl/))，所以測試一下在 ASP.NET Core 上啟用 gRPC Reflection 是否有改善

## 基本環境

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.4(46911)
3. .NET Core SDK 3.1.301
4. 預設 gRPC Service 專案範本
5. NuGet packages

    - Grpc.AspNetCore.Server.Reflection 2.31.0

## 設定方式

1. 安裝 NuGet package - `Grpc.AspNetCore.Server.Reflection`

    - .NET CLI

        ```bash
        dotnet add package Grpc.AspNetCore.Server.Reflection
        ```

    - Package Manager

        ```bash
        Install-Package Grpc.AspNetCore.Server.Reflection
        ```

2. 註冊 gRPC reflection

    > 在 `Startup.cs` 的 `ConfigureServices` 方法中進行註冊

    ```cs
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddGrpc();
        //註冊 gRPC reflection
        services.AddGrpcReflection();
    }
    ```

3. 設定 gRPC reflection 的 mapping

    > 在 `Startup.cs` 的 `Configure` 方法中進行設定

    ```cs
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapGrpcService<GreeterService>();
            // 設定啟用 gRPC reflection
            endpoints.MapGrpcReflectionService();

            endpoints.MapGet("/",
                async context =>
                {
                    await context.Response.WriteAsync(
                        "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
                });
        });
    }
    ```

4. 實際使用

    - 語法

        ```bash
        grpcurl -plaintext localhost:5000 list
        ```

    - 範例

        ```bash
        grpcurl -plaintext localhost:5000 list
        ```

    - 未設定啟用 reflection 錯誤

        - 錯誤訊息

            ```txt
            Failed to list services: server does not support the reflection API
            ```

        - 錯誤截圖

            ![1error](https://user-images.githubusercontent.com/3851540/93022110-b769c900-f619-11ea-8b35-eed64d748e30.png)

    - 成功設定結果

        - 成功訊息

            ```txt
            greet.Greeter
            grpc.reflection.v1alpha.ServerReflection
            ```

        - 成功截圖

            ![3success](https://user-images.githubusercontent.com/3851540/93022112-b89af600-f619-11ea-823a-5f6fb1c0c41f.png)

## 心得

設定過程很輕鬆寫意，但如果是以一開始需求的出發點(不想手 key proto 檔)來看，在效率的改善上有限，原因是就 grpcurl 的使用方式而言僅能取得 grpc service 而已，無法取得相對應的 message 資訊：grpcurl 透過 reflection 取得 service 名稱，但個別 grpc service 用的 message 還是得要查閱 proto 檔才能正確呼叫 grpc service，不過這個問題可以搭配其他 cli 工具或是選用其他適合的工具，如果有興趣可以參考其他筆記

- [使用 grpc-cli 呼叫 gRPC Service](/grpc-cli/)
- [使用 dotnet-grpc-cli 取得 gRPC Service 內容](/dotnet-grpc-cli/)

## 參考資訊

1. [使用grpcurl 呼叫gRPC Service](/grpcurl/)
2. [gRPC Server Reflection in the .NET world](https://martinbjorkstrom.com/posts/2020-07-08-grpc-reflection-in-net)
3. [使用 grpc-cli 呼叫 gRPC Service](/grpc-cli/)
4. [使用 dotnet-grpc-cli 取得 gRPC Service 內容](/dotnet-grpc-cli/)
