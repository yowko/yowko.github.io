---
title: "使用 grpcurl 使用 Timestamp 參數呼叫 gRPC Service"
date: 2020-03-09T23:30:00+08:00
lastmod: 2020-03-09T23:30:31+08:00
draft: false
tags: ["gRPC","Tools"]
slug: "grpcurl-timestamp"
---

## 使用 grpcurl 使用 Timestamp 參數呼叫 gRPC Service

之前筆記 [使用 grpcurl 呼叫 gRPC Service](https://blog.yowko.com/grpcurl/) 紀錄到使用 `grpcurl` 就可以不用 gui 工具以及自行撰寫程式來呼叫 gRPC Service，正以為可以順利解決問題時發現有些使用情境跟想像中不同，趕緊筆記一下備忘

## 基本環境說明

1. macOS Catalina 10.15.3
2. grpcurl v1.4.0
3. 測試用 proto

    > 這邊需要特別留意 `timestamp.proto` 的引用方式，以及 `ProtoRoot` 的路徑設定

    ```txt
    syntax = "proto3";

    option csharp_namespace = "GrpcMessages";

    package greet;

    import "Google/timestamp.proto";

    // The greeting service definition.
    service Greeter {
      // Sends a greeting
      rpc SayHello (HelloRequest) returns (HelloReply);
    }

    // The request message containing the user's name.
    message HelloRequest {
      string name = 1;
      google.protobuf.Timestamp RequestTime = 2;
    }

    // The response message containing the greetings.
    message HelloReply {
      string message = 1;
    }
    ```

## 實際使用狀況

1. 依 `timestamp` 定義指定：`seconds`,`nanos`

    ```bash
    grpcurl -d '{"name":"Yowko","RequestTime":{"seconds":1583715969,"nanos":0}}' -plaintext -import-path /Users/yowko.tsai/POCs/GrpcMessages/Protos -proto greet.proto localhost:5000 greet.Greeter/SayHello
    ```

2. 錯誤訊息

    - 訊息內容

        ```txt
        Error invoking method "greet.Greeter/SayHello": error getting request data: json: cannot unmarshal object into Go value of type string
        ```

    - 錯誤截圖

        ![1error](https://user-images.githubusercontent.com/3851540/76177031-cc3e4d80-61ed-11ea-8df9-74cdea5283db.png)

3. 解決方式：直接傳遞 `RFC 3339` 時間格式

    > 據 [Use RFC 3339 string representation of google.protobuf.Timestamp](https://github.com/uw-labs/bloomrpc/issues/98#issuecomment-510736760) 可以直接使用 `RFC 3339` 時間格式即可，使用方式與 `bloomrpc` 指定 `seconds`,`nanos` 不同

    - 使用特定時區

        ```bash
        grpcurl -d '{"name":"Yowko","RequestTime":"2009-06-15T13:45:30.0000000+08:00"}' -plaintext -import-path /Users/yowko.tsai/POCs/GrpcMessages/Protos -proto greet.proto localhost:5000 greet.Greeter/SayHello

        ```

    - 使用 UTC 時區

        ```bash
        grpcurl -d '{"name":"Yowko","RequestTime":"2009-06-15T13:45:30.00Z"}' -plaintext -import-path /Users/yowko.tsai/POCs/GrpcMessages/Protos -proto greet.proto localhost:5000 greet.Greeter/SayHello
        ```

## 心得

我覺得 `grpcurl` 的用法 (使用 `RFC 3339` 時間格式) 比較方便也相對直覺，只是因為之前主要使用 `bloomrpc` 的 `seconds`,`nanos` 設定方式而有的既定印象

當時看到錯誤訊息：`json: cannot unmarshal object into Go value of type string` google 一些資料後沒找到正確解法，為了避免日後使用又遇到相同問題，簡單筆記一下

## 參考資訊

1. [使用 grpcurl 呼叫 gRPC Service](https://blog.yowko.com/grpcurl/)
2. [Use RFC 3339 string representation of google.protobuf.Timestamp](https://github.com/uw-labs/bloomrpc/issues/98#issuecomment-510736760)
