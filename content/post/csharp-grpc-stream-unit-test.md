---
title: "嘗試為gRPC 中的 stream RPC 加上 Unit Test"
date: 2019-06-19T21:30:00+08:00
lastmod: 2019-06-19T21:30:31+08:00
draft: false
tags: ["csharp","gRPC","Unit Test"]
slug: "csharp-grpc-stream-unit-test"
---

# 嘗試為gRPC 中的 stream RPC 加上 Unit Test

之前筆記 [C# 搭配 gRPC 中使用 stream RPC](https://blog.yowko.com/csharp-grpc-stream/) 紀錄到在 gRPC 中使用 stream RPC 的操作語法，但實際應用在專案上時卻卡關，主因是單元測試出現錯誤，仔細看了錯誤原因才發現 stream RPC 雖然只是在 service 的參數定義加上 `stream` 不過 generate 出來的實作方法中則用了不同的參數型別，儘管經過嘗試後有找到方法不過仍然有缺點，於是先筆記目前的做法，待日後有機會再看看有沒有其他更漂亮的方式

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet package

    - Grpc 1.21.0
    - Grpc.Tools 1.21.0
    - Google.Protobuf 3.8.0
    - Bogus 27.0.1
4. protobuf 定義

    > 大致上內容延續之前筆記 [C# 搭配 gRPC 中使用 stream RPC](https://blog.yowko.com/csharp-grpc-stream/) 內容，多加上 server-side streaming RPC 回傳多筆資料

    - message

        ```proto
        syntax = "proto3";

        package Message; //will be placed in a namespace matching the package name if csharp_namespace is not specified
        option csharp_namespace = "GRpc.Messages";

        message Candidates {
            repeated Candidate Candidates = 2;
        }

        message Candidate {
            string Name = 1;
            repeated Job Jobs = 2;
        }

        message Job {
            string Title = 1;
            int32 Salary = 2;
            string JobDescription = 3;
        }

        message DownloadByName {
            string Name = 1;
        }

        message CreateCvResponse {
            bool IsSuccess = 1;
        }
        ```

    - service

        ```proto
        syntax = "proto3";

        package Message; //will be placed in a namespace matching the package name if csharp_namespace is not specified
        option csharp_namespace = "GRpc.Messages";
        import "message.proto";
        import "Google/empty.proto";

        service CandidateService {
            rpc CreateCv (stream Candidate) returns (CreateCvResponse);
            rpc DownloadCv (DownloadByName) returns (stream Candidate);
            rpc DownloadAllCv (google.protobuf.Empty) returns (stream Candidate);
            rpc CreateDownloadCv (stream Candidate) returns (stream Candidates);
        }
        ```

## 主機端串流 RPC (server-side streaming RPC)

1. 建立 `TestServerStreamWriter` 實作 `IServerStreamWriter`

    > 其中 `WriteOptions` 與 `WriteAsync` 是來自於 `IServerStreamWriter` 的底層 `IAsyncStreamWriter<T>`

    ```cs
    private class TestServerStreamWriter<T> : IServerStreamWriter<T>
    {
        public WriteOptions WriteOptions { get; set; }

        public List<T> Responses { get; } = new List<T>();

        public Task WriteAsync(T message)
        {
            this.Responses.Add(message);

            return Task.CompletedTask;
        }
    }
    ```

2. 測試程式

    > 建立 `TestServerStreamWriter` 實體，並當做實作方法的參數傳入以接受資料

    - 取得一筆

        ```cs
        [Test]
        public async Task DownloadCv_ShouldGetOne()
        {
            // arrange
            var target = new CandidateServiceImpl();
            const int expected = 1;
            var actual = new TestServerStreamWriter<Candidate>();
            var request = new DownloadByName()
            {
                Name = "test"
            };

            // act
            await target.DownloadCv(request, actual, null);

            // assert
            actual.Responses.Count.Should().Be(expected);
        }
        ```

    - 取得多筆

        ```cs
        [Test]
        public async Task DownloadAllCv_ShouldGetFive()
        {
            // arrange
            var target = new CandidateServiceImpl();
            const int expected = 5;
            var actual = new TestServerStreamWriter<Candidate>();
            var request = new Empty();

            // act
            await target.DownloadAllCv(request, actual, null);

            // assert
            actual.Responses.Count.Should().Be(expected);
        }
        ```

## 用戶端串流 RPC (client-side streaming RPC)

1. 建立 `TestAsyncStreamReader` 實作 `IAsyncStreamReader<T>`

    > 其中 `Current` 與 `MoveNext` 來自  `IAsyncStreamReader` 的底層 `IAsyncEnumerator<out T>`

    ```cs
    private class TestAsyncStreamReader<T> : IAsyncStreamReader<T>
    {
        public TestAsyncStreamReader(T message)
        {
            this.Current = message;
        }

        public void Dispose()
        {
        }

        private bool _hasNext = true;

        public Task<bool> MoveNext(CancellationToken cancellationToken)
        {
            var result = Task.FromResult(_hasNext);
            _hasNext = false;

            return result;
        }

        public T Current { get; private set; }
    }
    ```

2. 測試程式

    > 建立 `TestAsyncStreamReader` 實體，並當做實作方法的參數傳入以接受資料

    ```cs
    [Test]
    public async Task CreateCv_ShouldGetTrue()
    {
        // arrange
        var target = new CandidateServiceImpl();

        // 做假資料 start
        var fakeJobs = new Faker<Job>()
            .RuleFor(a => a.Title, (f, u) => f.Company.Bs())
            .RuleFor(a => a.Salary, (f, u) => f.Commerce.Random.Int(1000, 2000))
            .RuleFor(a => a.JobDescription, (f, u) => f.Lorem.Text());

        var createRequests = new Faker<Candidate>()
            .RuleFor(a => a.Name, (f, u) => f.Name.FullName())
            .RuleFor(a => a.Jobs, (f, u) =>
            {
                u.Jobs.AddRange(fakeJobs.GenerateBetween(3, 5));

                return u.Jobs;
            }).Generate();
        // 做假資料 end

        var reader = new TestAsyncStreamReader<Candidate>(createRequests);

        // act
        var actual = await target.CreateCv(reader, null);


        // assert
        actual.Should().NotBeNull();
        actual.IsSuccess.Should().BeTrue();
    }
    ```

## 雙向串流 RPC (bidirectional streaming RPC)

1. 建立 `TestServerStreamWriter` 實作 `IServerStreamWriter` (如果之前已經建立過不需重新建立)

    > 其中 `WriteOptions` 與 `WriteAsync` 是來自於 `IServerStreamWriter` 的底層 `IAsyncStreamWriter<T>`

    ```cs
    private class TestServerStreamWriter<T> : IServerStreamWriter<T>
    {
        public WriteOptions WriteOptions { get; set; }

        public List<T> Responses { get; } = new List<T>();

        public Task WriteAsync(T message)
        {
            this.Responses.Add(message);

            return Task.CompletedTask;
        }
    }
    ```

2. 建立 `TestAsyncStreamReader` 實作 `IAsyncStreamReader<T>` (如果之前已經建立過不需重新建立)

    > 其中 `Current` 與 `MoveNext` 來自  `IAsyncStreamReader` 的底層 `IAsyncEnumerator<out T>`

    ```cs
    private class TestAsyncStreamReader<T> : IAsyncStreamReader<T>
    {
        public TestAsyncStreamReader(T message)
        {
            this.Current = message;
        }

        public void Dispose()
        {
        }

        private bool _hasNext = true;

        public Task<bool> MoveNext(CancellationToken cancellationToken)
        {
            var result = Task.FromResult(_hasNext);
            _hasNext = false;

            return result;
        }

        public T Current { get; private set; }
    }
    ```

3. 程式程式

    > 建立 `TestAsyncStreamReader` 與 `TestServerStreamWriter` 來當做測試資料的容器

    ```cs
    [Test]
    public async Task CreateDownloadCv_ShouldGetFive()
    {
        // arrange
        var target = new CandidateServiceImpl();

        // 假造資料 start
        var fakeJobs = new Faker<Job>()
            .RuleFor(a => a.Title, (f, u) => f.Company.Bs())
            .RuleFor(a => a.Salary, (f, u) => f.Commerce.Random.Int(1000, 2000))
            .RuleFor(a => a.JobDescription, (f, u) => f.Lorem.Text());

        var createRequests = new Faker<Candidate>()
            .RuleFor(a => a.Name, (f, u) => f.Name.FullName())
            .RuleFor(a => a.Jobs, (f, u) =>
            {
                u.Jobs.AddRange(fakeJobs.GenerateBetween(3, 5));

                return u.Jobs;
            }).Generate();
        // 假造資料 end
        var reader = new TestAsyncStreamReader<Candidate>(createRequests);
        var writer = new TestServerStreamWriter<Candidates>();

        var expected = new Candidates();
        expected.Candidates_.Add(createRequests);

        // act
        await target.CreateDownloadCv(reader, writer, null);


        // assert
        writer.Should().NotBeNull();
        writer.Responses.Should().BeEquivalentTo(expected);
    }
    ```

## 心得

為什麼筆記標題是 `嘗試`，主要是因為雖然透過繼承 stream 相關 interface，但方法或是屬性上還是與實作用法有些差異，而在 gRPC-dotnet 官方的 GitHub 相關測試也是使用同樣方式來進行，不知道是我沒看完整還是目前暫時只能這麼做

其中我覺得做法上有重大缺陷的是 `雙向串流 RPC (bidirectional streaming RPC)` 中的 ServerStreamWriter 無法傳回多筆資料，原因不太好說明，大意是 `StreamReader` 與 `ServerStreamWriter` 都是 mock 出來的，沒有實際進行 stream 傳送與接放也就無法逐次取得 object
 
## 參考資訊

1. [C# 搭配 gRPC 中使用 stream RPC](https://blog.yowko.com/csharp-grpc-stream/)
2. [WrappedClientStreamWriter](https://github.com/grpc/grpc-dotnet/blob/072a616b5a6a4250410f37f41128f0fbd8b2805b/test/Grpc.Net.Client.Tests/InterceptorTests.cs#L159)
3. [TestServerStreamWriter](https://github.com/grpc/grpc-dotnet/blob/bd453582ccba40134aa8fcf0eefd7fdfd8c53ad5/test/Grpc.AspNetCore.Server.Tests/Reflection/ReflectionGrpcServiceActivatorTests.cs#L82)
4. [HttpContextStreamWriter](https://github.com/grpc/grpc-dotnet/blob/61de5a0305c7705d2dfc19905a8699191c16f4f2/src/Grpc.AspNetCore.Server/Internal/HttpContextStreamWriter.cs#L25)
