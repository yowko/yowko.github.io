---
title: "在 macOS 上的 ASP.NET Core 中呼叫 gRPC"
date: 2020-02-23T22:30:00+08:00
lastmod: 2020-02-23T22:30:31+08:00
draft: false
tags: ["macOS","dotnet core","gRPC"]
slug: "macos-aspnetcore-grpc"
---

## 在 macOS 上的 ASP.NET Core 中呼叫 gRPC

最近為了進行某個專案需求的 poc，需要建立基本的 gRPC Server 與 Cient，這才想到之前都是在 .NET Core 2 上使用 gRPC，還沒紀錄過 .NET Core 3 的 gRPC 使用方式 (所以沒得抄XD)，想說趁這個機會順手紀錄一下，這下不得了  沒想到讓我卡了一陣子：搞得好像怎麼設定都是錯的，一直出現錯誤，後來重新在 Windows 驗證一次，確定是 macOS 上特有的 feature，所以更應該紀錄一下，順便等日後 Microsoft 會不會做些優化，可以做個對照

## 基本環境說明

1. macOS Catalina 10.15.3
2. .NET Core SDK 3.1.102
3. 專案結構與 NuGet 套件

    - GpcService (gRPC Service 預設專案範本)
      - Grpc.AspNetCore 2.27.0
    - GrpcMessages (.NET Core class library - 用來存放 *.proto)
      - Google.Protobug 3.11.4
      - Grpc.Net.Client 2.270.0
      - Grpc.Tools 2.27.0
    - GrpcClient (ASP.NET Core MVC 預設專案範本)
      - Grpc.AspNetCore 2.27.0

## 設定方式

1. GrpcService

    > 這個部份可以參考之前筆記 [ASP.NET Core gRPC 無法在 macOS 上啟動？！](https://blog.yowko.com/aspdotnet-core-grpc-macos/) 或是 [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos)，主因就是 gRPC 預設使用 TLS，但 macOS 不支援具有 TLS 的 ASP.NET Core gRPC 所以需要在 Grpc Server 端設定停用 TLS ：修改 `Program.cs` 的 `CreateHostBuilder` 方法

    ```cs
    public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                options.ListenLocalhost(5000, o => o.Protocols = HttpProtocols.Http2);
            });
            webBuilder.UseStartup<Startup>();
        });
    ```

2. GrpcClient

    > 需要將 `System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` 參數設定為 `true` 才能允許呼叫沒有 TLS 加密的 gRPC Server：修改 `Startup.cs` 中的 `ConfigureServices` 方法

    ```cs
    public void ConfigureServices(IServiceCollection services)
        {

            services.AddSingleton(a =>
            {
                AppContext.SetSwitch(
                    "System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);

                var channel = GrpcChannel.ForAddress("http://localhost:5000");
                var client = new Greeter.GreeterClient(channel);
                return client;
            });
            services.AddControllers();
        }
    ```

## 未正確設定可能遭遇的錯誤

1. Server 端未停用 TLS; Client 也未允許不加密傳輸

    - 錯誤訊息

        ```txt
        RpcException: Status(StatusCode=Internal, Detail="Request protocol 'HTTP/1.1' is not supported.")
        ```

    - 錯誤截圖

        ![1noservernoclient](https://user-images.githubusercontent.com/3851540/75629012-6401d180-5c19-11ea-9196-c8ad2d34f8fb.png)

2. Server 未停用； Client 允許不加密傳輸

    - 錯誤訊息

        ```txt
        RpcException: Status(StatusCode=Internal, Detail="Error starting gRPC call: An error occurred while sending the request.")
        ```

    - 錯誤截圖

        ![2onlyclient](https://user-images.githubusercontent.com/3851540/75629013-682def00-5c19-11ea-9b2f-371f21fe43eb.png)

3. server 停用； Client 未允許不加密傳輸

    - 錯誤訊息

        ```txt
        RpcException: Status(StatusCode=Internal, Detail="Error starting gRPC call: Connection refused")
        ```

    - 錯誤截圖

        ![3onlyserver](https://user-images.githubusercontent.com/3851540/75629014-695f1c00-5c19-11ea-87d9-f7b65fcc5c6c.png)

## 心得

對照一下 Windows 上的寫法 [在 Windows 上的 ASP.NET Core 中呼叫 gRPC](https;//blog.yowko.com/windows-aspnetcore-grpc)，還是覺得 macOS 上的這些設定很多餘，不僅讓程式碼變多、複雜、也讓程式碼需要有處理不同環境而走不同 flow 的能力，我感覺起來是變醜滿多的，但牽扯到 os level 的架構問題，似乎也是沒什麼調整空間，不過我還是對 Microsoft 有信心，總覺得過陣子會有不錯的解決方案出來的

## 參考資訊

1. [教學課程：在 ASP.NET Core 中建立 gRPC 用戶端和伺服器](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-3.1&tabs=visual-studio&WT.mc_id=DOP-MVP-5002594)
2. [使用 .NET Core 用戶端呼叫不安全的 gRPC 服務](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594#call-insecure-grpc-services-with-net-core-client)
3. [ASP.NET Core gRPC 無法在 macOS 上啟動？！](https://blog.yowko.com/aspdotnet-core-grpc-macos/)
4. [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-3.1&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos)
5. [在 Windows 上的 ASP.NET Core 中呼叫 gRPC](https;//blog.yowko.com/windows-aspnetcore-grpc)
