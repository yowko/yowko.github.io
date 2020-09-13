---
title: "使用 dotnet-grpc-cli 取得 gRPC Service 內容"
date: 2020-09-13T12:30:00+08:00
lastmod: 2020-09-13T12:30:31+08:00
draft: false
tags: ["gRPC","dotnet"]
slug: "dotnet-grpc-cli"
---

## 使用 dotnet-grpc-cli 取得 gRPC Service 內容

之前筆記 [使用 grpc-cli 呼叫 gRPC Service](https://blog.yowko.com/grpc-cli/) 紀錄到 gprc 官方 command line tool 的使用方式，後來偶爾間發現竟然有 dotnet 版：dotnet-grpc-cli，還是 C# 撰寫的，立馬來嘗試看看囉

## 基本環境設定

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.4(46911)
3. dotnet-grpc-cli 0.2.0

## 安裝與使用

1. 安裝方式

    ```bash
    dotnet tool install -g dotnet-grpc-cli
    ```

2. 使用方式

    - 取得所有 service

        - 語法

            ```bash
            dotnet grpc-cli ls {grpc server endpoint}
            ```

        - 範例

            ```bash
            dotnet grpc-cli ls http://localhost:5000
            ```

            ![1ls](https://user-images.githubusercontent.com/3851540/93022064-6a85f280-f619-11ea-86ad-00f52baa0f65.png)

    - 取得指定 service 詳細資訊

        - 語法

            ```bash
            dotnet grpc-cli ls {grpc server endpoint} {service name} -l
            ```

        - 範例

            ```bash
            dotnet grpc-cli ls http://localhost:5000 greet.Greeter -l
            ```

            ![2lsservice](https://user-images.githubusercontent.com/3851540/93022066-6bb71f80-f619-11ea-9482-d8939ef36693.png)

    - 取得指定 method 詳細資訊

        - 語法

            ```bash
            dotnet grpc-cli ls {grpc server endpoint} {service.method name} -l
            ```

        - 範例

            ```bash
            dotnet grpc-cli ls http://localhost:5000 greet.Greeter.SayHello -l
            ```

            ![3lsmethod](https://user-images.githubusercontent.com/3851540/93022068-6c4fb600-f619-11ea-8ac5-ef6ad9dec3c9.png)

    - 以 proto 格式取得 service

        - 語法

            ```bash
            dotnet grpc-cli dump {grpc server endpoint} {service.method name}
            ```

        - 範例

            ```bash
            dotnet grpc-cli dump http://localhost:5000 greet.Greeter.SayHello
            ```

            ![4dump](https://user-images.githubusercontent.com/3851540/93022070-6ce84c80-f619-11ea-9cf9-d7fc5b3e6ef6.png)

## 心得

dotnet-grpc-cli 與 grpc-cli 

減少了：

1. 取得指定 message type 詳細資訊
2. 呼叫指定 method

增加了：

- Dump service in proto format
- 需要指明確指定 endpoint protocal

相同的是在沒有啟用 `Reflection` 下一樣無用武之地，只是錯誤不太一樣

![5error](https://user-images.githubusercontent.com/3851540/93022071-6d80e300-f619-11ea-8ee5-83f911f613ba.png)

除此之外，個人覺得 dotnet-grpc-cli 少了直接呼叫 grpc method 的方法，功能只是為了取得 grpc service 與 method 定義，在使用上太侷限了

## 參考資訊

1. [使用 grpc-cli 呼叫 gRPC Service](https://blog.yowko.com/grpc-cli/)
2. [dotnet-grpc-cli](https://www.nuget.org/packages/dotnet-grpc-cli/)
3. [mholo65/dotnet-grpc-cli](https://github.com/mholo65/dotnet-grpc-cli)
