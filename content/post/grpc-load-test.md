---
title: "關於 gRPC 的 Load Test"
date: 2022-04-29T00:30:00+08:00
lastmod: 2022-04-29T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp","gRPC","Benchmark"]
slug: "grpc-load-test"
---

## 關於 gRPC 的 Load Test

搜尋資訊的過程中，偶爾看到 [Load testing for gRPC - the case](https://alexromanov.github.io/2021/08/23/gatling-grpc/)，分析用來針對 gRPC service 做 load test 的三種方式：

1. Gatling with gRPC plugin.

    > - 使用 scala 來建立測試腳本
    > - 支援四種 gRPC service：一元(Unary Call)、client streaming、server streaming、雙向 streaming

2. k6.io.

    > - 使用 Javascript 來建立測試腳本
    > - 只支援 一元(Unary Call) gRPC SERVICE

3. ghz.

    > - CLI 工具，只要指定參數不用寫腳本
    > - 支援四種 gRPC service：一元(Unary Call)、client streaming、server streaming、雙向 streaming

## 實際測試

1. Gatling with gRPC plugin.

    詳細內容請參考之前筆記 [使用 Gatling 來對 gRPC 做負載測試](/grpc-load-test-gatling)

2. k6.io.

    詳細內容請參考之前筆記 [使用 k6 來對 gRPC 做負載測試](/grpc-load-test-k6)

3. ghz.

    詳細內容請參考之前筆記 [使用 ghz 來對 gRPC 做負載測試](/grpc-load-test-ghz)

## 心得

以下是個人觀點，請大家依照自身的實際情況進行評估

1. k6.io

    > 只能測試 一元(Unary call) gRPC service 的限制，讓我第一步就直接不考慮了

2. Gatling with gRPC plugin

    > 以我個人來看，進入門檻太高了，從環境設定、專案結構、套件 dependency、scala 語法、IDE 每個步驟對我都是困難重重，需要花時間一一克服，不是簡單的隨取隨用

3. ghz

    > 參數雖然很多，但至少不用重新了解一個開發語言，使用上也單純：下載對應平台的 binary 就可以直接執行，少了很多環境設定上的關卡

## 參考資訊

1. [Load testing for gRPC - the case](https://alexromanov.github.io/2021/08/23/gatling-grpc/)
2. [使用 Gatling 來對 gRPC 做負載測試](/grpc-load-test-gatling)
3. [使用 k6 來對 gRPC 做負載測試](/grpc-load-test-k6)
4. [使用 ghz 來對 gRPC 做負載測試](/grpc-load-test-ghz)
