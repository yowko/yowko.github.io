---
title: "Protobuf 時間屬性該如何表示？"
date: 2019-03-12T21:30:00+08:00
lastmod: 2021-10-28T21:30:31+08:00
draft: falae
tags: ["csharp","gRPC","Protobuf"]
slug: "protobuf-datetime-timestamp"
---
## Protobuf 時間屬性該如何表示？

最近的專案在跨 application 的溝通上捨去以往熟悉的 RESTful API 而採用 gRPC 做為溝通的 protocal，過去沒有相關使用經驗的我當然是踩雷不斷，不過也有種重新入門的新鮮感

以 C# 為例，最常用來描述時間格式就是 DateTime，但 gRPC 使用的 Protobuf 格式中並沒有 DateTime 的資料類型，需要 C# 做些轉換，今天就先紀錄 Protobuf 可以如何表示時間類型的資料

## 基本環境說明

1. macOS Mojave 10.14.3
2. Grpc 1.19.0
3. Grpc.Tools 1.19.0
4. Google.Protobuf 3.7.0
5. Google.Protobuf.Tools 3.7.0
6. 資料夾結構

    ```txt
    -- gRPC.Timestamp
        -- gRPC.Timestamp.sln
        -- proto
            -- message.proto
        -- src
            -- gRPC.Message (netstandard2.0)
            -- gRPC.Client (netcoreapp2.2)
            -- gRPC.Server (netcoreapp2.2)
    ```

7. gRPC.Message projcet
    - 安裝套件
        - gRPC
            - Package Manager

                ```bash
                Install-Package Grpc
                ```

            - .NET CLI

                ```bash
                dotnet add package Grpc
                ```

        - Google.Protobuf
            - Package Manager

                ```bash
                Install-Package Google.Protobuf
                ```

            - .NET CLI

                ```bash
                dotnet add package Google.Protobuf
                ```

    - 加入 model

        ```cs
        public class UserEntity
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public int Age { get; set; }
            public DateTime Birthday { get; set; }
        }
        ```

8. gRPC.Client 與 gRPC.Server 皆參考 gRPC.Message
9. gRPC.Client

    ```cs
    var host = "127.0.0.1";
    var port = "9999";


    var channel = new Channel($"{host}:{port}", ChannelCredentials.Insecure);

    var serviceClient = new gRPCService.gRPCServiceClient(channel);

    ```

10. gRPC.Server 實作 gRPCService 並啟動 gRPC
    - 實作

        ```cs
        public class gRPCServiceImplfor : gRPCService.gRPCServiceBase
        {
            public override Task<Response> AddUser(AddUserRequest request, ServerCallContext context)
            {
                return base.AddUser(request, context);
            }

            public override Task<Response> GetUsers(GetUsersRequest request, ServerCallContext context)
            {
                return base.GetUsers(request, context);
            }

            public override Task<Response> DeleteUser(DeleteUserRequest request, ServerCallContext context)
            {
                return base.DeleteUser(request, context);
            }
        }
        ```

    - 啟動

        ```cs
        static async Task Main(string[] args)
        {
            var host = "127.0.0.1";
            var port = 9999;

            var serverInstance = new Grpc.Core.Server
            {
                Ports =
                {
                    new ServerPort(host, port, ServerCredentials.Insecure)
                }
            };

            Console.WriteLine($"Demo server listening on host:{host} and port:{port}");

            serverInstance.Services.Add(
                Message.gRPCService.BindService(
                    new gRPCServiceImpl()));

            serverInstance.Start();

            Console.ReadKey();

            await serverInstance.ShutdownAsync();
        }
        ```

## 方法一：使用 int64

1. message 定義

    ```proto
    syntax = "proto3";

    package gRPC.Message; 
    option csharp_namespace = "gRPC.Message";
    
    message AddUserRequest{
        string Name=1;
        int32 Age=2;
        int64 Birthday=3;
    }

    ```

2. 實際使用

    - 發送端將 DateTime 轉為 ToUnixTimestamp

        ```cs
        new AddUserRequest
        {
            Name = "Yowko",
            Age = 35,
            Birthday = ((DateTimeOffset)new DateTime(1983, 7, 29)).ToUnixTimeSeconds()
        }
        ```

    - 接收端再轉回 DateTime

        ```cs
        new UserEntity
        {
            Id = 1,
            Name = request.Name,
            Age = request.Age,
            Birthday = DateTimeOffset.FromUnixTimeSeconds(request.Birthday).DateTime
        };
        ```

## 方法二：使用 Timestamp

1. message 定義

    > `Timestamp` 是 google 額外提供的型別，使用時需要 import

    ```proto
    syntax = "proto3";

    package gRPC.Message;
    option csharp_namespace = "gRPC.Message";
    import "google/protobuf/timestamp.proto";

    message AddUserRequest{
        string Name=1;
        int32 Age=2;
        google.protobuf.Timestamp Birthday=3;
    }
    ```

2. 編譯需額外引用參考 timestamp.proto

    > timestamp.proto 位於 `/Users/``whoami``/.nuget/packages/google.protobuf.tools/3.6.1/tools/google/protobuf/timestamp.proto`

    ```proto
    /Users/`whoami`/.nuget/packages/grpc.tools/1.18.0/tools/macosx_x64/protoc -I /Users/`whoami`/.nuget/packages/google.protobuf.tools/3.6.1/tools/ -I ./proto/ --csharp_out gRPC.Message --grpc_out gRPC.Message ./proto/*.proto --plugin=protoc-gen-grpc=/Users/`whoami`/.nuget/packages/grpc.tools/1.18.0/tools/macosx_x64/grpc_csharp_plugin
    ```

3. 實際使用

    - 傳送端

        ```cs
        new AddUserRequest
        {
            Name = "Yowko",
            Age = 35,
            Birthday =  Timestamp.FromDateTime(new DateTime(1983,7,29).ToUniversalTime())
        }
        ```

    - 接受端

        ```cs
        new UserEntity
        {
            Id = 1,
            Name = request.Name,
            Age = request.Age,
            Birthday = request.Birthday.ToDateTime()
        };
        ```

## 心得

以專案的角度來看，兩者都能解決問題 - 都是好方法，不過如果必需從中選出一個，我會選用 `Timestamp` ，主要原因是語意比較清楚，使用 int64 比較容易誤解為一段長數字

回到 protobuf，開發流程與過去一鍵完成的做法不同(需要手動引用、手動編譯、手動調整編譯參數)，不過就像一開始提到的重新學習不同的開發模式也是種 reset 自己的趣味，反正都體驗看看再來評估好壞囉

## 參考資訊

1. [gRPC development on .NET Core - Basic](https://blackie1019.github.io/2019/02/10/gRPC-development-on-NET-Core-Basic/)
2. [How to convert byte array to any type](https://stackoverflow.com/a/33022788)
