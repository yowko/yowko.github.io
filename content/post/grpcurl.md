---
title: "使用 grpcurl 呼叫 gRPC Service"
date: 2020-03-7T12:30:00+08:00
lastmod: 2020-03-07T12:30:31+08:00
draft: false
tags: ["gRPC","Tools"]
slug: "grpcurl"
---

## 使用 grpcurl 呼叫 gRPC Service

一般情況下測試 gRPC 服務，我大多是透過簡易的 console 直接呼叫 (一來可以順便檢查程式，二來 stream 相關功能比較齊全)，如果想要測試的功能只是 simple call 時我就會透過 [BloomRPC](https://appimage.github.io/BloomRPC/) 這套 gui 工具來呼叫

剛好前陣子開發用環境遇到問題，想要針對部署至 Kubernetes 中的服務進行測試、驗證功能，但服務同時啟動三個 pos 做 load balance，無法對每個 pod 上的服務做檢查，這時候就可以使用 grpcurl 來達成目的，下載 grpcurl 後，使用 grpcurl 透過每個 pod 的 cluster ip 來叫用服務，雖然使用難度不高，不過最近明顯感受到記憶力衰退，還是紀錄一下備忘，下次需要時就可以更快取用了

## 基本環境說明

1. CentOS 7.7
2. macOS Catalina 10.15.3
3. Debian 10
4. grpcurl v1.4.0
5. 測試用 proto

    ```txt
    syntax = "proto3";

    option csharp_namespace = "GrpcMessages";

    package greet;

    // The greeting service definition.
    service Greeter {
      // Sends a greeting
      rpc SayHello (HelloRequest) returns (HelloReply);
    }

    // The request message containing the user's name.
    message HelloRequest {
      string name = 1;
    }

    // The response message containing the greetings.
    message HelloReply {
      string message = 1;
    }
    ```

## 關於 grpcurl

grpcurl 是一個命令列工具，主要目的是從 terminal 中呼叫 gRPC Service

因為 gRPC 使用 protobuf 的關係，讓 `curl` 無法直呼叫 gRPC 服務，而 grpcurl 則使用 JSON 傳送訊息

grpcurl 在 gRPC Service 支援的前提下可以使用 `reflection` 來取得 gRPC Service 的 schema (簡言之就是透過 `list` 方法列出目前的 gRPC Service 有哪些方法與所需參數)，但這個方法需要調整到程式碼部份，我個人還沒有嘗試，我單純透過 gRPC Service 所使用的 proto 檔來呼叫 gRPC 方法

grpcurl 宣稱支援所有 gRPC 傳輸方式 (包含 streaming)，但我實際只用過 simple RPC 與 server-side streaming RPC，至於 client-side streaming RPC 與 bidirectional streaming RPC 暫時還沒測試

## 安裝方式

1. macOS

    ```bash
    brew install grpcurl
    ```

2. CentOS

    - 下載 grpcurl

        ```bash
        curl -L -O https://github.com/fullstorydev/grpcurl/releases/download/v1.4.0/grpcurl_1.4.0_linux_x86_64.tar.gz
        ```

    - 解壓 grpcurl

        ```bash
        tar -zxvf grpcurl_1.4.0_linux_x86_64.tar.gz
        ```

3. Debian (Ubuntu)

    - 更新 apt-get

        ```bash
        apt-get update
        ```

    - 安裝 wget

        ```bash
         apt-get install -y wget
        ```

    - 下載 grpcurl

        ```bash
        wget https://github.com/fullstorydev/grpcurl/releases/download/v1.4.0/grpcurl_1.4.0_linux_x86_64.tar.gz
        ```

    - 解壓 grpcurl

        ```bash
        tar -zxvf grpcurl_1.4.0_linux_x86_64.tar.gz
        ```

4. 透過 go 安裝 (source code)

    ```bash
    go get github.com/fullstorydev/grpcurl
    go install github.com/fullstorydev/grpcurl/cmd/grpcurl
    ```

## 使用方式

> 這邊我沒有額外調整 gRPC 的 flow (沒用到 reflection 功能)，只透過指定 proto 檔案的方式來呼叫 gRPC Service

- 語法範例

    ```bash
    grpcurl -d '{呼叫 method 的參數}' -import-path {proto 所在資料夾位置} -proto {proto 檔名} {gRPC-Server:port} {proto 檔中的 package name}.{service name}/{method name}
    ```

- 實際使用

    ```bash
    grpcurl -d '{"name":"Yowko"}' -import-path /Users/yowko.tsai/POCs/grpcurl/GrpcMessages/Protos -proto greet.proto localhost:5000 greet.Greeter/SayHello
    ```

## 可能遇到的錯誤

1. gRPC Service 未啟用 TLS

    - 錯誤訊息

        ```txt
        Failed to dial target host "localhost:5000": tls: first record does not look like a TLS handshake
        ```

    - 錯誤截圖

        ![1error1](https://user-images.githubusercontent.com/3851540/76146403-65297780-60cd-11ea-8e6c-92d500d19249.png)

    - 解決方式

        > 使用 `-plaintext` 方式來呼叫 gRPC

2. 參數格式錯誤

    - 錯誤訊息

        ```txt
        Error invoking method "greet.Greeter/SayHello": error getting request data: invalid character 'n' looking for beginning of object key string
        ```

    - 錯誤截圖

        ![2error2](https://user-images.githubusercontent.com/3851540/76146404-678bd180-60cd-11ea-9dbc-9716c10d662f.png)

    - 解決方式

        > 重新檢視 `-d` 所傳遞參數需為 `json` 格式，且前後由 `''` 包覆

## 心得

透過 `grpcurl` 工具就可以解決在沒有 gRPC gui 工具想要驗證 gRPC 內容的需求，而如同筆記一開始提過的 `grpcurl` 支援 gRPC 所有呼叫方式，也相對反應在 `grpcurl` 的參數上，可以設定的東西不少，不過很多參數我都沒有仔細了解，就是求個堪用、有辦法解決問題，或許慢慢再找時間試試透過 `grpcurl` 的各種呼叫方法與參數設定吧

## 參考資訊

1. [BloomRPC](https://appimage.github.io/BloomRPC/)
2. [fullstorydev/grpcurl](https://github.com/fullstorydev/grpcurl)
