---
title: "gRPC stream 如何傳送單一大物件 (Client 版)"
date: 2019-08-01T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["C#","gRPC"]
slug: "grpc-stream-chunk-client"
---

## gRPC stream 如何傳送單一大物件 (Client 版)

繼之前筆記 [gRPC stream 如何傳送單一大物件](/grpc-stream-big-object/) 提到該如何使用 gRPC stream 來傳送不是整齊 collection 物件後，公司專案已逐步將可能傳送超出預設 4mb 限制的 api 改用 stream 以避免可能出現的 `RESOURCE_EXHAUSTED` 錯誤，截至目前使用上沒有遇到異常狀況。

上班前同事問到，該如何透過 client 端來上傳單一大物件，我翻了翻筆記，發現沒得抄，想要直接開工卻想不起來該怎麼做 (看來真的不能不服老呀)，所以立馬來筆記一下，避免以後沒地方抄又想很久 QQ

今天內容會以之前筆記 [gRPC stream 如何傳送單一大物件](/grpc-stream-big-object/)  為基礎作延伸

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. NuGet Package

    - Google.Protobuf 3.8.0
    - Bogus 28.0.1

        > 產生假資料用 (非必要)

    - Grpc 1.22.0
    - Grpc.Tools 1.22.0
4. proto

    - message

        ```proto
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

        message Chunk {
            bytes chunk = 1;
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
            rpc GetFakePerson (google.protobuf.Empty) returns (stream Chunk);
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

1. service.proto 加入允許使用 stream 上傳的方式

    ```proto
    rpc CreatePerson (stream Chunk) returns (Person);
    ```

2. client 上傳大物件

     > protobuf 序列化的用法可以參考 [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](/csharp-protobuf-serialize-deserialize/)

    - 使用 `stream`
    - 透過 Google.Protobuf 轉為 byte[] 傳送

    ```cs
    using (var creator = serviceClient.CreatePerson())
    {
        var createPerson = GeneratePerson();

        using (var ms = new MemoryStream())
        {

            createPerson.WriteTo(ms);
            //每次以 64k 傳送
            const int chunkSize = 64 * 1024;
            var streamResult = ms.ToArray();
            var chunkCount = streamResult.Length % chunkSize == 0
                ? streamResult.Length / chunkSize
                : (streamResult.Length / chunkSize) + 1;

            for (var i = 0; i < chunkCount; i++)
            {
                await creator.RequestStream.WriteAsync( new Chunk(){Chunk_ = ByteString.CopyFrom(streamResult.Skip(chunkSize*i).Take(chunkSize).ToArray())} );
            }
        }
        //等待完成上傳
        await creator.RequestStream.CompleteAsync();
        //回傳 gRPC call 結果
        return await creator;
    }
    ```

3. server 端接收大物件

    > protobuf 反序列化的用法可以參考 [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](/csharp-protobuf-serialize-deserialize/)

    - 使用 stream 接收
    - 透過 Google.Protobuf 讀取 byte[] 轉型

    ```cs
    var streamResult = new List<byte>();
    //逐步接收內容
    while (await requestStream.MoveNext())
    {
        var current = requestStream.Current;
        streamResult.AddRange(current.Chunk_);
    }
    //將收到的 byte array 轉為 Person
    var person =  Person.Parser.ParseFrom(streamResult.ToArray());
    //回傳收到的內容
    return person;
    ```

## 心得

仔細看上面所使用的例子可以發現有個 bug，我們使用 gRPC stream 就是想要處理超乎 4mb 單次傳送限制的內容， client 上傳改用 srteam 這部份程式沒有問題，但回傳時卻沒有用上 stream，如果上傳的物件真的超過 4mb 時就會出現錯誤，不過我考慮再三還是決定先不加，因為這次筆記的重點是紀錄 client 上傳使用 stream chunk 的做法，如果加上回傳也用上 stream 就變成雙向 stream，跟這次目的不符也會使得程式碼變複雜，如果有需要雙向 stream 之後再另外筆記囉

## 參考資訊

1. [gRPC stream 如何傳送單一大物件](/grpc-stream-big-object/)
2. [C# 中使用 Protocol Buffers 協定來序列化與反序列化物件](/csharp-protobuf-serialize-deserialize/)
