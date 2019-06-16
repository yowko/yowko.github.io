---
title: "C# 搭配 gRPC 中使用 stream RPC"
date: 2019-06-16T21:30:00+08:00
lastmod: 2019-06-16T21:30:31+08:00
draft: false
tags: ["csharp","gRPC"]
slug: "csharp-grpc-stream"
---

# C# 搭配 gRPC 中使用 stream RPC

gRPC 允許使用四種則型的 service 方法：

1. 簡單 RPC (simple RPC)
2. 主機端串流 RPC (server-side streaming RPC)
3. 用戶端串流 RPC (client-side streaming RPC)
4. 雙向串流 RPC (bidirectional streaming RPC)

過去的筆記都是使用 `簡單 RPC (simple RPC)`，最近剛好工作上需要用到 streaming RPC，紀錄一下使用方式備忘

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet package

    - Grpc 1.21.0
    - Grpc.Tools 1.21.0
    - Google.Protobuf 3.8.0
    - Bogus 27.0.1
4. protobuf 定義

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

        service CandidateService {
            rpc CreateCv (Candidate) returns (CreateCvResponse);
            rpc DownloadCv (DownloadByName) returns (Candidate);
            rpc CreateDownloadCv (Candidate) returns (Candidates);
        }
        ```

## 實際使用 stream

今天就三種 streaming RPC 來紀錄 (simple RPC 之前筆記就一直使用中，不特別紀錄了)

1. 主機端串流 RPC (server-side streaming RPC)

    > 適合從 server 端取得較大資料量時使用

    - 在 service 的 rpc 定義中將 return 的 message 加上 `stream` 修飾子

        ```proto
        rpc DownloadCv (DownloadByName) returns (stream Candidate);
        ```

    - `Server` 實作 stream 傳送

        ```cs
        public override async Task DownloadCv(DownloadByName request, IServerStreamWriter<Candidate> responseStream,
            ServerCallContext context)
        {
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
            // 將每筆資料逐一透過 WriteAsync 輸出
            await responseStream.WriteAsync(createRequests);
        }
        ```

    - `Client` 實作 stream 接收

        ```cs
        using (var client = candidateServiceClient.DownloadCv(new DownloadByName()
        {
            Name = "test"
        }))
        {
            // 逐一取出 stream 內容
            while (await client.ResponseStream.MoveNext())
            {
                result = client.ResponseStream.Current;
            }
        }
        ```

2. 用戶端串流 RPC (client-side streaming RPC)

    > 適合傳送較大資料量至 server 端時使用

    - 在 service 的 rpc 定義中將 rpc 接受的 message 加上 `stream` 修飾子

        ```proto
        rpc CreateCv (stream Candidate) returns (CreateCvResponse);
        ```

    - `Server` 實作 stream 讀取

        ```cs
        public override async Task<CreateCvResponse> CreateCv(IAsyncStreamReader<Candidate> requestStream, ServerCallContext context)
        {
            var result = new CreateCvResponse
            {
                IsSuccess = false
            };
            // stream 讀取
            while (await requestStream.MoveNext())
            {
                var candidate = requestStream.Current;
                // 實際處理
                Console.WriteLine(candidate.Name);
            }

            return result;
        }
        ```

    - `Client` 實作 stream 傳送

        ```cs
        using (var creator = candidateServiceClient.CreateCv())
        {
            // 將資料逐一輸出至 server
            foreach (var createRequest in createRequests)
            {
                await creator.RequestStream.WriteAsync(createRequest);
            }

            await creator.RequestStream.CompleteAsync();

            var summary = await creator.ResponseAsync;
        }
        ```

3. 雙向串流 RPC (bidirectional streaming RPC)

    > 適合在 client 與 server 端間雙向傳送大資料量或是即時資料時使用

    - 在 service 的 rpc 定義中將 rpc 傳送與接受的 message 都加上 `stream` 修飾子

        ```proto
        rpc CreateDownloadCv (stream Candidate) returns (stream Candidates);
        ```

    - `Server` 實作 stream

        ```cs
        public override async Task CreateDownloadCv(IAsyncStreamReader<Candidate> requestStream,
            IServerStreamWriter<Candidates> responseStream, ServerCallContext context)
        {
            var candidates = new Candidates();
            // 將收到的資料逐一取出
            while (await requestStream.MoveNext())
            {
                var candidate = requestStream.Current;
                candidates.Candidates_.Add(candidate);
                // 將處理後的資料回傳
                await responseStream.WriteAsync(candidates);
            }
        }
        ```

    - `Client` 實作 stream 

        ```cs
        using (var call = candidateServiceClient.CreateDownloadCv())
        {
            var responseReaderTask = Task.Run(async () =>
            {
                // 逐一取出 response 內容
                while (await call.ResponseStream.MoveNext())
                {
                    var candidate = call.ResponseStream.Current;
                    result.AddRange(candidate.Candidates_);
                }
            });

            // 將資料逐一傳送至 server
            foreach (var request in createRequests)
            {
                await call.RequestStream.WriteAsync(request);
            }

            await call.RequestStream.CompleteAsync();
            await responseReaderTask;
        }
        ```

## 心得

資料量大時透過 stream 可以使用即時做出回應而不需等待所有資料傳送完成，程式碼並不多也滿好理解的，先紀錄一下使用方式，後續再紀錄實際使用情境及其他相關用法

完整程式碼請參考 [yowko/dotnetgrpcstream](https://github.com/yowko/dotnetgrpcstream)

## 參考資訊

1. [gRPC Basics - C#](https://grpc.io/docs/tutorials/basic/csharp/)
2. [yowko/dotnetgrpcstream](https://github.com/yowko/dotnetgrpcstream)
