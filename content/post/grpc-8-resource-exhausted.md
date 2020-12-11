---
title: "gRPC 出現 `8 RESOURCE_EXHAUSTED` 錯誤"
date: 2019-06-23T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["csharp","gRPC"]
slug: "grpc-8-resource-exhausted"
---

## gRPC 出現 `8 RESOURCE_EXHAUSTED` 錯誤

隨著系統一步步成形，資料量也愈來愈大，在原本只是先求功能正常而未進行資料分頁的功能逐漸露出原型，今天就來筆記 gRPC 在傳送龐大資料可能會遇到的錯誤以及解決方式

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. NuGet package

    - Grpc 1.21.0
    - Grpc.Tools 1.21.0
    - Google.Protobuf 3.8.0
    - Bogus 27.0.1

        > 建立假資料用

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
        import "Google/empty.proto";

        service CandidateService {
            rpc CreateCvSimple (Candidate) returns (CreateCvResponse);
            rpc DownloadCvSimple (google.protobuf.Empty) returns (Candidate);
            rpc CreateCv (stream Candidate) returns (CreateCvResponse);
            rpc DownloadCv (DownloadByName) returns (stream Candidate);
            rpc DownloadAllCv (google.protobuf.Empty) returns (stream Candidate);
            rpc DownloadAllCvOneTime (google.protobuf.Empty) returns (stream Candidates);
            rpc CreateDownloadCv (stream Candidate) returns (stream Candidates);
        }
        ```

## 錯誤訊息

1. 訊息內容

    ```text
    {
        "error": "8 RESOURCE_EXHAUSTED: Received message larger than max (4479822 vs. 4194304)"
    }
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/59977181-91b16800-9600-11e9-9e86-a17fd179ea15.png)

## 解決方式

1. 調整 gRPC 傳送訊息大小限制

    > 調整接受端的 `MaxReceiveMessageLength`，預設傳送端的 `MaxSendMessageSize` 為 `Int32.MaxValue`，預設接受端則為 `4194304` (4MB)

    - Server

        > 上傳大量資料時調整 Server, 以下示範調整為預設值的兩倍

        ```cs
        new Grpc.Core.Server(
                new List<ChannelOption>
                {
                    new ChannelOption(ChannelOptions.MaxReceiveMessageLength, 4194304 * 2)
                }
        ```

    - Client

        > 取回大量資料時調整 Client,以下示範調整為預設值的三倍

        ```cs
        services.AddSingleton(
                new CandidateService.CandidateServiceClient(
                    new Channel(HostString, ChannelCredentials.Insecure
                        , new List<ChannelOption>()
                        {
                            new ChannelOption(ChannelOptions.MaxReceiveMessageLength, 4194304 * 3)
                        }
                    )));
        ```

2. 改用 stram RPC

    > 詳細使用方式可以參考之前筆記  [C# 搭配 gRPC 中使用 stream RPC](/csharp-grpc-stream/) , 大意是在 gRPC service 定義時在需要大量資料的參數加上 `stream` 修飾子，但需要留意的是如果單一批量的訊息大小超過 `4194304 - 4MB` 時，還是需要修改 `MaxReceiveMessageLength` 或是降低批量大小

    - client-side streaming RPC

        > 上傳大量資料

    - server-side streaming RPC

        > 下載大量資料

    - bidirectional streaming RPC

        > 上傳及下載大量資料，或是即時傳輸

## 心得

上述兩種方式都可以解決 message 內容過大的問題，建議視情境來選擇，不過一般來說放寬訊息大小限制意謂著傳輸時間會變長，也連帶讓處理與回應時間變長，並不是非常符合現在普遍講究快速回應的系統目標，而 stream 則是在收到第一個批量時即可進行處理而不需等待所有訊息內容接受完畢，讓呼叫端收到第一個回應的時間點可以較早，但針對批量資料的彙總類數值就失去了這個好處

## 參考資訊

1. [Getting “Error: 8 RESOURCE_EXHAUSTED: Sent message larger than max (2217 vs. 15)” from hyperledger fabric](https://stackoverflow.com/questions/50293910/getting-error-8-resource-exhausted-sent-message-larger-than-max-2217-vs-15)
2. [帶入gRPC：gRPC Streaming, Client and Server](https://segmentfault.com/a/1190000016503114)
