---
title: "gRPC stream 如何傳送單一大物件"
date: 2019-07-07T21:30:00+08:00
lastmod: 2019-07-07T21:30:31+08:00
draft: false
tags: ["csharp","gRPC"]
slug: "grpc-stream-big-object"
---

## gRPC stream 如何傳送單一大物件

之前筆記 [C# 搭配 gRPC 中使用 stream RPC](https://blog.yowko.com/csharp-grpc-stream/) 提到為了對於較大資料量以及即時性資料內容，可以透過 gRPC 的 stream RPC 來處理，不過官方範例是用在傳送 `repeated` 內容 (如同 List、Array 這類的物件)，但現實上難免會遇到需要傳送不只一個 list 的狀況：像是一個大型物件中，包含多個不同長度的 list，剛好專案就有類似需求，我就小小筆記一下，當作備忘

今天內容會以之前筆記 [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](https://blog.yowko.com/csharp-protobuf-serialize-deserialize/) 為基礎作延伸

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet Package

    - Google.Protobuf 3.8.0
    - Bogus 28.0.1

        >  產生假資料用 (非必要)
    - Grpc 1.22.0
    - Grpc.Tools 1.22.0

4. proto

    - message

        ```ptoto
        syntax = "proto3";

        package Messages; //will be placed in a namespace matching the package name if csharp_namespace is not specified
        option csharp_namespace = "Messages";

        import "timestamp.proto";

        message Person{
            string Id=1;
            google.protobuf.Timestamp Birthday=2;
            string Name=3;
            repeated Job jobs=4;
            repeated Certificate Certificates=5;
            repeated School Schools=6;
        }

        message Job{
            string CompanyName=1;
            string JobTitle=2;
            google.protobuf.Timestamp DateFrom=3;
            google.protobuf.Timestamp DateTo=4;
        }

        message Certificate{
            string Name=1;
            string IssueOrg=2;
            google.protobuf.Timestamp IssueDate=3;
        }

        message School{
            string SchoolName=1;
            bool IsGraduated=2;
            google.protobuf.Timestamp DateFrom=3;
            google.protobuf.Timestamp DateTo=4;
        }
        ```

    - service

        ```proto
        syntax = "proto3";

        package Messages; //will be placed in a namespace matching the package name if csharp_namespace is not specified
        option csharp_namespace = "Messages";
        import "message.proto";
        import "empty.proto";

        service TestService {
            rpc GetFakePerson (google.protobuf.Empty) returns (Person);
        }
        ```

5. 使用 Bogus 產生假資料

    ```cs
    static Person GeneratePerson()
    {
        var certificates = new Faker<Certificate>()
            .RuleFor(a => a.Name, (f, u) => f.Random.Word())
            .RuleFor(a => a.IssueDate, f=>f.Date.Past().ToUniversalTime().ToTimestamp())
            .RuleFor(a => a.IssueOrg, (f, u) => f.Company.CompanyName()).Generate(3);

        var jobs= new Faker<Job>()
            .RuleFor(a=>a.CompanyName,(f, u) =>f.Company.CompanyName())
            .RuleFor(a=>a.JobTitle,(f, u) =>f.Person.Company.Bs)
            .RuleFor(a=>a.DateFrom,(f, u) => f.Date.Past(3).ToUniversalTime().ToTimestamp())
            .RuleFor(a=>a.DateTo,(f, u) =>f.Date.Past(1).ToUniversalTime().ToTimestamp())
            .Generate(4);
        var schools = new Faker<School>()
            .RuleFor(a=>a.SchoolName,(f, u) =>f.Company.CompanyName())
            .RuleFor(a=>a.DateFrom,(f, u) =>f.Date.Past(3).ToUniversalTime().ToTimestamp())
            .RuleFor(a=>a.DateTo,(f, u) => f.Date.Past(1).ToUniversalTime().ToTimestamp())
            .RuleFor(a=>a.IsGraduated,(f,u)=>f.Random.Bool()).Generate(5)
            ;
        var person = new Faker<Person>()
            .RuleFor(a=>a.Name,(f,u)=>f.Person.FullName)
            .RuleFor(a=>a.Id,(f,u)=>Guid.NewGuid().ToString())
            .RuleFor(a=>a.Birthday,(f,u)=>f.Person.DateOfBirth.ToUniversalTime().ToTimestamp())
            .Generate()
            ;

        person.Certificates.AddRange(certificates);
        person.Jobs.AddRange(jobs);
        person.Schools.AddRange(schools);
        return person;
    }
    ```

## 修改方式

因為無法將一個大物件中做合理分群 (像是 list 切分為多個小 lsit) 進行 stream 傳送，所以作法改為 `傳送 byte[]` 這樣一樣就可以有效對於 stream rpc 要傳送的內容進行有群切分了

1. 加入 field 為 `bytes` 的 message

    ```proto
    message Chunk {
        bytes chunk = 1;
    }
    ```

2. 原本回傳大物件的 service 改為 `stream Chunk`

    ```proto
    rpc GetFakePerson (google.protobuf.Empty) returns (stream Chunk);
    ```

3. 修改傳送端

    > protobuf 序列化的用法可以參考 [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](https://blog.yowko.com/csharp-protobuf-serialize-deserialize/)

    - 改用 `stream`
    - 透過 Google.Protobuf 轉為 byte[] 傳送

    ```cs
    var result = GeneratePerson();
    using (var ms = new MemoryStream())
    {

        result.WriteTo(ms);
        //每次以 64k 傳送
        const int chunkSize = 64 * 1024;
        var streamResult = ms.ToArray();
        var chunkCount = streamResult.Length % chunkSize == 0
            ? streamResult.Length / chunkSize
            : (streamResult.Length % chunkSize) + 1;

        for (var i = 0; i < chunkCount; i++)
        {
            await responseStream.WriteAsync( new Chunk(){Chunk_ = ByteString.CopyFrom(streamResult.Skip(chunkSize*i).Take(chunkSize).ToArray())} );

        }
    }
    ```

4. 修改接受端

    > protobuf 反序列化的用法可以參考 [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](https://blog.yowko.com/csharp-protobuf-serialize-deserialize/)

    - 改用 stream 接收
    - 透過 Google.Protobuf 讀取 byte[] 轉型

    ```cs
    var streamResult = new List<byte>();
    using (var client = serviceClient.GetFakePerson(new Empty()))
    {
        // 逐一取出 stream 內容
        while (await client.ResponseStream.MoveNext())
        {
            streamResult.AddRange(client.ResponseStream.Current.Chunk_.ToByteArray());
        }
    }

    var person = Person.Parser.ParseFrom(streamResult.ToArray());

    return person;
    ```

## 心得

雖然依上述步驟可以解決 `使用 stream rpc 傳送單一大型物件` 的問題，但做法上我自己覺得還有改進空間

1. stream RPC 需要自行決定傳送型態與切割傳送大小
2. client 端需先暫存 byte[]，官方文件提到 Google.Protobuf 可以直接讀取 ByteString

    > 這個可能是我自己沒找到方法

## 參考資訊

1. [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](https://blog.yowko.com/csharp-protobuf-serialize-deserialize/)
2. [Chunking large messages with gRPC](https://jbrandhorst.com/post/grpc-binary-blob-stream/)
