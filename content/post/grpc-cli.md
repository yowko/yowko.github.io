---
title: "使用 grpc-cli 呼叫 gRPC Service"
date: 2020-09-12T12:30:00+08:00
lastmod: 2021-01-08T12:30:31+08:00
draft: false
tags: ["gRPC"]
slug: "grpc-cli"
---

## 使用 grpc-cli 呼叫 gRPC Service

之前筆記 [使用grpcurl 呼叫gRPC Service](/grpcurl/) 紀錄到 grpcurl (curl for grpc) 的使用方式，最近在查其他資料時這才發現原來 gRPC 官方也有提供：grpc-cli，想不到我如此後知後覺！當然要立馬測試、紀錄，以茲驚惕

## 基本環境設定

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.4(46911)
3. grpc 1.31.1

## 安裝與使用

1. 安裝方式

    ```bash
    brew tap grpc/grpc
    brew install grpc
    ```

2. 使用方式

    - 取得所有 service

        - 語法

            ```bash
            grpc_cli ls {grpc server endpoint}
            ```

        - 範例

            ```bash
            grpc_cli ls localhost:5000
            ```

            ![1lsservice](https://user-images.githubusercontent.com/3851540/93022035-2eeb2880-f619-11ea-982e-36204a8be62b.png)

    - 取得指定 service 詳細資訊

        - 語法

            ```bash
            grpc_cli ls {grpc server endpoint} {service name} -l
            ```

        - 範例

            ```bash
            grpc_cli ls localhost:5000 greet.Greeter -l
            ```

            ![2lsservicel](https://user-images.githubusercontent.com/3851540/93022036-301c5580-f619-11ea-831a-a0238f462e30.png)

    - 取得指定 method 詳細資訊

        - 語法

            ```bash
            grpc_cli ls {grpc server endpoint} {service.method name} -l
            ```

        - 範例

            ```bash
            grpc_cli ls localhost:5000 greet.Greeter.SayHello -l
            ```

            ![3lsmethod](https://user-images.githubusercontent.com/3851540/93022037-314d8280-f619-11ea-8d29-e5cc2a92102c.png)

    - 取得指定 message type 詳細資訊

        - 語法

            ```bash
            grpc_cli type {grpc server endpoint} {message type name}
            ```

        - 範例

            ```bash
            grpc_cli type localhost:5000 greet.HelloRequest
            ```

            ![4lsmessagetype](https://user-images.githubusercontent.com/3851540/93022038-31e61900-f619-11ea-8b0c-be15224900d6.png)

    - 呼叫指定 method

        - 語法

            ```bash
            grpc_cli call {grpc server endpoint} {service.mathod name} {request message}
            ```

        - 範例

            ```bash
            grpc_cli call localhost:5000 SayHello "name: 'gRPC CLI'"
            ```

            ![5call](https://user-images.githubusercontent.com/3851540/93022039-31e61900-f619-11ea-9d86-15336494c8bc.png)

## 心得

初步使用起來，在啟用 `Reflection` 下個人覺得比 grpcurl 來得便利：可以直接從 grpc server 端就能取得完整 service、method、request message 內容，只是前提是必需啟用 `Reflection`，如果未啟用 `Reflection` 就連基本的 method call 都會有問題

![6error](https://user-images.githubusercontent.com/3851540/93022040-327eaf80-f619-11ea-82bb-8400352f368f.png)

## 參考資訊

1. [使用grpcurl 呼叫gRPC Service](/grpcurl/)
2. [brew grpc](https://formulae.brew.sh/formula/grpc)
3. [homebrew-grpc](https://github.com/grpc/homebrew-grpc)
4. [grpc/doc/command_line_tool.md](https://github.com/grpc/grpc/blob/master/doc/command_line_tool.md)
