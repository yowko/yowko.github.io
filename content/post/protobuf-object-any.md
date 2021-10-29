---
title: "Protobuf 該如何處理不定型別"
date: 2019-03-16T21:30:00+08:00
lastmod: 2021-10-28T21:30:31+08:00
draft: falae
tags: ["csharp","gRPC","Protobuf"]
slug: "protobuf-object-any"
---
## Protobuf 該如何處理不定型別

之前筆記 [Protobuf 時間屬性該如何表示？](/protobuf-datetime-timestamp/) 紀錄了 C# DateTime 屬性在 Protobuf 的 message 表示方式，當時在找資料時發現 `any.proto` 特別查了資料看可以應用在什麼地方，就個人理解應該就像是 C# 的 object，筆記一下用法，待日後驗證囉

## 基本環境說明

1. macOS Mojave 10.14.3
2. Grpc 1.19.0
3. Grpc.Tools 1.19.0
4. Google.Protobuf 3.7.0
5. Google.Protobuf.Tools 3.7.0
6. Bogus 26.0.1
7. Newtonsoft.Json 12.0.1
8. 資料夾結構

    ```txt
    -- gRPC.Any
        -- gRPC.Any.sln
        -- proto
            -- message.proto
        -- src
            -- gRPC.Message (netstandard2.0)
            -- gRPC.Client (netcoreapp2.2)
            -- gRPC.Server (netcoreapp2.2)
    ```

9. gRPC.Message projcet
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
        public class UserModel
        {
            public Guid Id { get; set; }
            public string Name { get; set; }
            public int Age { get; set; }
        }
        ```

10. gRPC.Client 與 gRPC.Server 皆參考 gRPC.Message
11. gRPC.Client

    ```cs
    var host = "127.0.0.1";
    var port = "9999";


    var channel = new Channel($"{host}:{port}", ChannelCredentials.Insecure);

    var serviceClient = new gRPCService.gRPCServiceClient(channel);

    ```

12. gRPC.Server 實作 gRPCService 並啟動 gRPC
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

## 方法一：使用 string

1. message 定義

    ```proto
    syntax = "proto3";

    package gRPC.Message;
    option csharp_namespace = "gRPC.Message";
    
    message AddUserRequest{
        string Name=1;
        int32 Age=2;
    }
    message GetUsersRequest{
    }

    message DeleteUserRequest{
        string Name=1;
    }

    message Response{
        bool IsSuccess=1;
        string ResultMsg=2;
        epeated string ResultMsgs=3;
    }


    service gRPCService {
        rpc AddUser(AddUserRequest) returns (Response);
        rpc GetUsers(GetUsersRequest) returns (Response);
        rpc DeleteUser(DeleteUserRequest) returns (Response);
    }
    ```

2. 實際使用

    - client 端

        ```cs
        static void Main(string[] args)
        {
            var host = "127.0.0.1";
            var port = "9999";


            var channel = new Channel($"{host}:{port}", ChannelCredentials.Insecure);

            var serviceClient = new gRPCService.gRPCServiceClient(channel);

            var result = serviceClient.AddUser(new AddUserRequest
            {
                Name = "Yowko",
                Age = 35
            });

            var addResult = JsonConvert.DeserializeObject<UserModel>(result.ResultMsg);
            Console.WriteLine($"Id:{addResult.Id};Name:{addResult.Name};Age:{addResult.Age}");

            var usersResult = serviceClient.GetUsers(new GetUsersRequest());

            var users = usersResult.ResultMsgs.Select(a => JsonConvert.DeserializeObject<UserModel>(a));


            foreach (var user in users)
            {
                Console.WriteLine($"Id:{user.Id};Name:{user.Name};Age:{user.Age}");
            }


            var deleteResult = serviceClient.DeleteUser(new DeleteUserRequest()
            {
                Name = "Yowko"
            });

            var delResult = deleteResult.ResultMsg;
            Console.WriteLine($"Msg:{delResult}");
        }
        ```

    - server 端

        ```cs
        public class gRPCServiceImpl : gRPCService.gRPCServiceBase
        {
            public override Task<Response> AddUser(AddUserRequest request, ServerCallContext context)
            {
                var user = FakeUserRule().Generate();

                user.Age = request.Age;
                user.Name = request.Name;
                var response = new Response
                {
                    IsSuccess = true,
                    ResultMsg = JsonConvert.SerializeObject(user)
                };

                return Task.FromResult(response);
            }

            public override Task<Response> DeleteUser(DeleteUserRequest request, ServerCallContext context)
            {
                var response = new Response
                {
                    IsSuccess = true,
                    ResultMsg = "User has deleted !!"
                };

                return Task.FromResult(response);
            }

            public override Task<Response> GetUsers(GetUsersRequest request, ServerCallContext context)
            {
                var response = new Response
                {
                    IsSuccess = true
                };
                var users = FakeUserRule().Generate(3);
                response.ResultMsgs.AddRange(users.Select(a => JsonConvert.SerializeObject(a))
                );

                return Task.FromResult(response);
            }

            private static Faker<UserModel> FakeUserRule()
            {
                return new Faker<UserModel>()
                    .RuleFor(a => a.Id, b => Guid.NewGuid())
                    .RuleFor(a => a.Name, (f, u) => f.Name.FirstName())
                    .RuleFor(a => a.Age, f => f.Random.Number(1, 10));
            }
        }
        ```

## 方法二：使用 any

1. message 定義

    >`any` 是 google 額外提供的型別，使用時需要 import

    ```proto
    syntax = "proto3";

    package gRPC.Message;
    option csharp_namespace = "gRPC.Message";
    import "google/protobuf/any.proto";

    message AddUserRequest{
        string Name=1;
        int32 Age=2;
    }
    message GetUsersRequest{
    }

    message DeleteUserRequest{
        string Name=1;
    }

    message Response{
        bool IsSuccess=1;
        google.protobuf.Any ResultMsg=2;
        repeated google.protobuf.Any ResultMsgs=3;
    }


    service gRPCService {
        rpc AddUser(AddUserRequest) returns (Response);
        rpc GetUsers(GetUsersRequest) returns (Response);
        rpc DeleteUser(DeleteUserRequest) returns (Response);
    }
    ```

2. 將 UserModel 加上 Serializable

    ```cs
    [Serializable]
    public class UserModel
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }
    ```

3. 在 gRPC.Message 中加入 ByteStringUtility

    > Any 型別使用 ByteString 來傳遞資料，加入 ByteStringUtility 來處理轉型

    ```cs
    public static class ByteStringUtility
    {
        public static byte[] ToByteArray<T>(T obj)
        {
            if(obj == null)
                return null;
            BinaryFormatter bf = new BinaryFormatter();
            using(MemoryStream ms = new MemoryStream())
            {
                bf.Serialize(ms, obj);
                return ms.ToArray();
            }
        }

        public static T FromByteArray<T>(byte[] data)
        {
            if(data == null)
                return default(T);
            BinaryFormatter bf = new BinaryFormatter();
            using(MemoryStream ms = new MemoryStream(data))
            {
                object obj = bf.Deserialize(ms);
                return (T)obj;
            }
        }
    }
    ```

4. 編譯需額外引用參考 any.proto

    > any.proto 位於 `/Users/``whoami``/.nuget/packages/google.protobuf.tools/3.7.0/tools/google/protobuf/any.proto`

    ```bash
    /Users/`whoami`/.nuget/packages/grpc.tools/1.19.0/tools/macosx_x64/protoc -I /Users/`whoami`/.nuget/packages/google.protobuf.tools/3.7.0/tools/ -I proto/  --csharp_out src/gRPC.Message --grpc_out src/gRPC.Message proto/*.proto --plugin=protoc-gen-grpc=/Users/`whoami`/.nuget/packages/grpc.tools/1.19.0/tools/macosx_x64/grpc_csharp_plugin
    ```

5. 實際使用

    - client 端

        ```cs
        var result = serviceClient.AddUser(new AddUserRequest
        {
            Name = "Yowko",
            Age = 35
        });

        var addResult =
            ByteStringUtility.FromByteArray<UserModel>(result.ResultMsg.Value
                .ToByteArray());
        Console.WriteLine($"Id:{addResult.Id};Name:{addResult.Name};Age:{addResult.Age}");

        var usersResult = serviceClient.GetUsers(new GetUsersRequest());

        var users = usersResult.ResultMsgs.Select(a =>
            ByteStringUtility.FromByteArray<UserModel>(a.Value.ToByteArray()));


        foreach (var user in users)
        {
            Console.WriteLine($"Id:{user.Id};Name:{user.Name};Age:{user.Age}");
        }
        

        var deleteResult = serviceClient.DeleteUser(new DeleteUserRequest()
        {
            Name = "Yowko"
        });

        var delResult = ByteStringUtility.FromByteArray<string>(deleteResult.ResultMsg.Value
            .ToByteArray());
        Console.WriteLine($"Msg:{delResult}");
        ```

    - server 端

        ```cs
        public class gRPCServiceImpl : gRPCService.gRPCServiceBase
        {
            public override Task<Response> AddUser(AddUserRequest request, ServerCallContext context)
            {
                var user = FakeUserRule().Generate();

                user.Age = request.Age;
                user.Name = request.Name;
                var response = new Response
                {
                    IsSuccess = true,
                    ResultMsg = new Any
                    {
                        Value = Google.Protobuf.ByteString.CopyFrom(ByteStringUtility.ToByteArray(user))
                    }
                };
                
                return Task.FromResult(response);
            }

            public override Task<Response> DeleteUser(DeleteUserRequest request, ServerCallContext context)
            {
                var response = new Response
                {
                    IsSuccess = true,
                    ResultMsg = new Any
                    {
                        Value = Google.Protobuf.ByteString.CopyFrom(ByteStringUtility.ToByteArray("User has deleted !!"))
                    }
                };

                return Task.FromResult(response);
            }

            public override Task<Response> GetUsers(GetUsersRequest request, ServerCallContext context)
            {
                var response = new Response
                {
                    IsSuccess = true
                };
                var users = FakeUserRule().Generate(3);
                response.ResultMsgs.AddRange(users.Select(a => new Any()
                {
                    Value = Google.Protobuf.ByteString.CopyFrom(ByteStringUtility.ToByteArray(a))
                }));

                return Task.FromResult(response);
            }

            private static Faker<UserModel> FakeUserRule()
            {
                return new Faker<UserModel>()
                    .RuleFor(a => a.Id, b => Guid.NewGuid())
                    .RuleFor(a => a.Name, (f, u) => f.Name.FirstName())
                    .RuleFor(a => a.Age, f => f.Random.Number(1, 10));
            }
        }
        ```

## 心得

以結果來看 string 省事不少，但直覺上應該是 any 效率比較好，不過口說無憑，改天有時間再來效能評測一下

回到 any 的使用情境，原本想試試可不可以達成 C# Generic 用途，但看來應該就是 object 而已，不過也是試過才知道，至少知道哪些可以做到哪些不行

程式碼可以參考 [yowko/gRPC-Any-CSharp-Sample](https://github.com/yowko/gRPC-Any-CSharp-Sample.git)

## 參考資訊

1. [gRPC development on .NET Core - Basic](https://blackie1019.github.io/2019/02/10/gRPC-development-on-NET-Core-Basic/)
2. [How to convert byte array to any type](https://stackoverflow.com/a/33022788)
3. [yowko/gRPC-Any-CSharp-Sample](https://github.com/yowko/gRPC-Any-CSharp-Sample.git)
