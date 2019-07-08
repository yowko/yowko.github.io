---
title: "C# 中使用 Protocol Buffers 協定來序列化與反序列化物件"
date: 2019-07-06T21:30:00+08:00
lastmod: 2019-07-06T21:30:31+08:00
draft: false
tags: ["csharp","protobuf"]
slug: "csharp-protobuf-serialize-deserialize"
---

## C# 中使用 Protocol Buffers 協定來序列化與反序列化物件

專案上剛好需要將 object 進行序列化，過去常用的方式都是序列為 `json`，後來同事覺得既然都使用 protobuf 了，為什麼不直接使用 protobuf 進行序列化就好？！ 我這才晃然大悟，想起 protobuf 本來就是用來做序列化與反序列化的嘛XD  因為沒有實際使用經驗，於是我來筆記一下，為我的愚蠢做個紀錄

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

## 安裝 NuGet 套件：`Google.Protobuf`

- Package Manager

    ```txt
    Install-Package Google.Protobuf
    ```

- .NET CLI

    ```bash
    dotnet add package Google.Protobuf
    ```

## 序列化 (Serialize)

> 需加入 `using Google.Protobuf;`

1. byte[]

    ```cs
    private static byte[] GetByteArray(Person person)
    {
        using (var ms = new MemoryStream())
        {
            person.WriteTo(ms);
            return ms.ToArray();
        }
    }
    ```

2. ByteString

    ```cs
    private static ByteString GetByteString(Person person)
    {
        using (var ms = new MemoryStream())
        {
            person.WriteTo(ms);
            return ByteString.CopyFrom(ms.ToArray());
        }
    }
    ```

3. Stream

     > 序列化動作正常，~~但無法正確反序列化回原物件~~  經強大同事提點已可正常運作

    ```cs
    private static MemoryStream GetStream(Person person)
    {
        var ms = new MemoryStream();
        person.WriteTo(ms);
        return ms;
    }
    ```

## 反序列化 (Deserialize)

1. byte[]

    ```cs
    private static Person GetPersonFromByteArray(byte[] bytes)
    {
        return Person.Parser.ParseFrom(bytes);
    }
    ```

2. ByteString

    ```cs
    private static Person GetPersonFromByteString(ByteString byteString)
    {
        return Person.Parser.ParseFrom(byteString);
    }
    ```

3. Stream

    > ~~無法正確反序列化回原物件~~  經強大同事提點已可正常運作

    ```cs
    private static Person GetPersonFromStream(Stream ms)
    {
        ms.Seek(0, SeekOrigin.Begin);
        return Person.Parser.ParseFrom(ms);
    }
    ```

## 心得

網路上多數透過 Protocol Buffer 進行序列化與反序列化的文章都是使用 `protobuf-net` NuGet 套件，一開始我也嘗試了一下：的確 `protobuf-net` 可以達成目的，不過需要在打算進行序列化的目標 class 上加入 `[ProtoContract]` 與 `[ProtoMember]` 才能進行序列化 (這動作與使用內建 `System.Runtime.Serialization` 加入 `[Serializable]` 相同)，雖然可以理解，不過使用上並不太便利

1. protobuf 的 message 生成的 .cs 不該自行修改，就算強行修改，下次重新 build，修改也會不見

    > 未設定的錯誤

    ```txt
    Unhandled Exception: System.InvalidOperationException: Type is not expected, and no contract can be inferred
    ```

2. class 常常有不同繼承關係，如果每層 class 都要手動加上 attibute，有些不切實際

所以我改用 `Google.Protobuf` 後就不需要再手動加入 `[ProtoContract]` 與 `[ProtoMember]` 也不用擔心漏加或是重新 build 後忘了加回去

另外官方提到 Serialize 與 Deserialize 可以透過 `stream`、`byte[]` 與 `ByteString`，不過 `stream` 我一直無法成功：我傳入 MemoryStream 進行 Deserialize 時，只能得到預設物件、一直無法取得正確資料，但這個問題我在 `protobuf-net` 上也有遇到，不知道是什麼原因不能這麼直接用，改天待發現根本原因，或是有大大願意提點時我再來註記

`2019/07/08 update  經同事說明 序列化到 stream 後要把 stream cursor 往回調才能解，確認可行，感謝強大同事指導`

## 參考資訊

1. [Convert any object to a byte array](https://stackoverflow.com/a/4865134/3600583)
2. [Protocol Buffer Basics: C#](https://developers.google.com/protocol-buffers/docs/csharptutorial)
