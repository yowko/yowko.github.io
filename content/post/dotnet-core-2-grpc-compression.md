---
title: "C# (.NET Core 2) 啟用 gRPC 壓縮"
date: 2019-11-16T21:30:00+08:00
lastmod: 2019-11-16T21:30:31+08:00
draft: false
tags: ["csharp","gRPC"]
slug: "dotnet-core-2-grpc-compression"
---

## C# (.NET Core 2) 啟用 gRPC 壓縮

目前專案在大資料量傳遞時會透過 gRPC stream，不過因為是非對稱式資料內容，採用 chunk byte 來傳輸，以避免單次 gRPC 的 message size 限制問題，但以當前的資料結構，資料量還是過大，導致處理時間無法有效縮短，同事提到想要使用壓縮來縮小資料量以減少傳輸的成本以及時間，不過測試下來似乎沒有減少傳輸時間，懷疑是壓縮沒有順利啟用

為了查明到底是 gRPC 壓縮沒有成功啟用，還是有其他原因造成傳輸時間沒有減少，就得先開啟 gRPC 的 tracer，詳細做法可以參考之前筆記 [C# (.NET Core 2) Log 與 Trace gRPC](https://blog.yowko.com/dotnet-core-2-log-grpc)

而今天就是來紀錄一下，gRPC 啟用壓縮的做法

## 基本環境說明

1. macOS Majave 10.14.16
2. .NET Core SDK 2.2.301
3. JetBrains Rider 2019.2.2
4. ASP.NET Core MVC 2.2 預設專案範本

    > 程式碼部份可參考 [C# 搭配 gRPC 中使用 stream RPC](https://blog.yowko.com/csharp-grpc-stream/)

5. NuGet Package

    - Google.Protobuf 3.8.0
    - Grpc 2.25.0
    - Grpc.Tools 2.25.0
    - BenchmarkDotNet 0.12.0

6. 未壓縮下 message 大於 4194304 (4mb)

    > 使用重複資料 * 5000 當做單次 message 內容

    ```txt
    Grpc.Core.RpcException: Status(StatusCode=ResourceExhausted, Detail="Received message larger than max (4860007 vs. 4194304)")
        at Grpc.Core.Internal.ClientResponseStream`2.MoveNext(CancellationToken token) in T:\src\github\grpc\src\csharp\Grpc.Core\Internal\ClientResponseStream.cs:line 60
        at Grpc.Core.Utils.AsyncStreamExtensions.ToListAsync[T](IAsyncStreamReader`1 streamReader) in T:\src\github\grpc\src\csharp\Grpc.Core\Utils\AsyncStreamExtensions.cs:line 49
        at GRpc.Client.Controllers.CandidateController.GetAllCandidate() in /Users/yowko.tsai/DotnetGrpcStream/GRpc.Client/Controllers/CandidateController.cs:line 162
    ```

    ![1error](https://user-images.githubusercontent.com/3851540/69007514-7f521500-0979-11ea-95c0-e6a964bbffb8.png)

## 啟用 gRPC 壓縮

1. 做法一： 使用 `gzip` 壓縮

    - 設定方式

        > 在單一 gRPC service 中設定

        ```cs
        public override async Task DownloadAllCv(Empty request, IServerStreamWriter<Candidate> responseStream,ServerCallContext context)
        {
            //設定使用 gzip 開始
            var compressionMetadata = new Metadata
            {
                {new Metadata.Entry("grpc-internal-encoding-request", "gzip")}
            };
            await context.WriteResponseHeadersAsync(compressionMetadata);
            //設定使用 gzip 結束

            foreach (var candidate in createRequests)
            {
                await responseStream.WriteAsync(candidate);
            }
        }
        ```

    - 實際效果

        > 可以看到 grpc 的 log 已經顯示出 `Compressed[gzip]`，以本次模擬的重複資料內容而言節省了 `99.76%` 的傳輸空間

        ```txt
        I1117 18:25:10.405264 123145570623488 /tmpfs/src/github/grpc/workspace_csharp_ext_macos_x64/src/core/ext/filters/http/message_compress/message_compress_filter.cc:265: Compressed[gzip] 4050007 bytes vs. 9888 bytes (99.76% savings)
        ```

        ![2gzip](https://user-images.githubusercontent.com/3851540/69007515-7f521500-0979-11ea-8fcf-28228dda5795.png)

2. 做法二： 使用 `deflate` 壓縮

    - 設定方式

        > 設定 gRPC Channel 的壓縮比 (這會套用至所有 gRPC service)

        ```cs
        var serverInstance = new Grpc.Core.Server(
            // 啟用 deflate 壓縮開啟
            new[] {new ChannelOption("grpc.default_compression_level", (int) CompressionLevel.High)}
            // 啟用 deflate 壓縮結束
            )
            {
                Ports =
                {
                    new ServerPort(host, port, ServerCredentials.Insecure)
                }
            };
        ```

    - 實際效果

        > 可以看到 grpc 的 log 已經顯示出 `Compressed[deflate]`，以本次模擬的重複資料內容而言節省了 `99.76%` 的傳輸空間

        ```txt
        I1117 18:44:54.079455 123145484414976 /tmpfs/src/github/grpc/workspace_csharp_ext_macos_x64/src/core/ext/filters/http/message_compress/message_compress_filter.cc:265: Compressed[deflate] 4050007 bytes vs. 9876 bytes (99.76% savings)
        ```

        ![3deflate](https://user-images.githubusercontent.com/3851540/69007516-7feaab80-0979-11ea-8308-1f7709e171af.png)

## 效能測試比較

- 第一次

    |   Method |     Mean |   Error |   StdDev |
    |---------  |---------:|--------:|---------:|
    | Gzip | 323.6 ms | 6.36 ms | 9.72 ms |
    | Deflate | 306.6 ms | 6.04 ms | 11.92 ms |

- 第二次

    |   Method |     Mean |    Error |   StdDev |
    |--------- |---------:|---------:|---------:|
    | Gzip | 376.1 ms | 14.12 ms | 41.40 ms |
    | Deflate | 347.4 ms | 6.94 ms | 19.81 ms |

- 第三次

    |   Method |     Mean |    Error |   StdDev |
    |--------- |---------:|---------:|---------:|
    | Gzip | 311.7 ms | 6.16 ms | 12.29 ms |
    | Deflate | 332.4 ms | 8.95 ms | 25.53 ms |

## 心得

以測試結果來看，互有勝負，沒有絕對哪一個方法比較好，若以使用方式來看 `gzip` 需要逐一在每個 service 上加入設定， `deflate` 則是一次全套用；看似 `deflate` 比較方便，但 [gRPC compression in C#](https://stackoverflow.com/questions/49031763/grpc-compression-in-c-sharp) 也提到啟用壓縮不一定會讓回應時間變快，在壓縮的處理也有一定的成本，到底該不該壓縮或是壓縮比例都需要經過測試的，為了符合不同情境，也可以在使用 `deflate` 針對特定 service 停用壓縮 - 透過在 service 中加上  `responseStream.WriteOptions = new WriteOptions(WriteFlags.NoCompress);`

花了不少時間測試，終於找到 gRPC 的 log 與壓縮的正確設定方式，相對應的做法在 .NET Core 3 會簡單不少，如果沒有特別的考量，建議還是升級 .NET Core 3 吧

## 參考資訊

1. [gRPC compression in C#](https://stackoverflow.com/questions/49031763/grpc-compression-in-c-sharp)
2. [C# (.NET Core 2) Log 與 Trace gRPC](https://blog.yowko.com/dotnet-core-2-log-grpc)
