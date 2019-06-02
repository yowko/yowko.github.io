---
title: "Kafka Producer 不同壓縮方式對發送速度的影響"
date: 2019-06-02T19:30:00+08:00
lastmod: 2019-06-02T19:30:31+08:00
draft: false
tags: ["Kafka","dotnet core"]
slug: "dotnet-kafka-producer-compresstype"
---

# Kafka Producer 不同壓縮方式對發送速度的影響

這兩天在測試 Kafka consumer 各項設定對於處理速度的影響 (詳細內容可以參考 [Kafka Consumer 處理速度緩慢，設定調整紀實](https://blog.yowko.com/dotnet-kafka-consumer-setting))，經過一輪比較後發現實際影響有限，最重要的還是需要降低 message 內容大小，在不調整發送流程、訊息內容的情況下，想要達到降低的 message 內容大小的做法就是對 message 進行壓縮了，只是壓縮不會是萬能藥對速度一定會有影響，就看影響的幅度多寡了

不過口說無憑，就來比較看看 Kafka 支援的幾種壓縮方式間有什麼差異吧

原本想用 BenchmarkDotNet 跑數據，但跑了一天一夜最後把 kafka 用的 volume 撐爆XD，暫時就用簡單方式大概了解趨勢囉

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. Docker version 18.09.2, build 6247962
4. NuGet package

    - Confluent.Kafka 1.0.1
    - ~~BenchmarkDotNet 0.11.5~~

5. Kafak 環境建置使用 [wurstmeister/kafka-docker](https://github.com/wurstmeister/kafka-docker)
6. 情境說明

    > 透過 docker-compose 啟動 kafka 以及 zookeeper，接著使用 console application 傳送 10萬則內容為 20267 字的 message

## CompressionType

Confluent.Kafka 支援四種壓縮方式(加上不壓縮為五種)

1. None

    > 不壓縮

2. Gzip

    > 1992 年就發表，也因為發展較早是常見的壓縮方式

3. Snappy

    > 過去稱 Zippy，是 Google 於 2011 年 open source，目標是高速與合理壓縮比

4. Lz4

    > 一種無失真資料壓縮演算法，追求速度，而非壓縮比

5. Zstd

    > Apache Kafka 2.1.0 開始支援 ZStandard - ZStandard 是 Facebook open source 的壓縮演算法，可以提供超高的壓縮比(compression ratio)

## 實際數據

> 以下數據單位皆為 `毫秒(Milliseconds)`

測試序\壓縮類型|None|Gzip|Snappy|Lz4|Zstd
:---|:---|:---|:---|:---|:---
1|158,630|150,031|130,903|132,211|159,511
2|140,655|134,789|116,066|116,722|143,775
3|131,750|133,232|116,756|115,459|142,409
4|150,077|136,454|121,711|122,406|143,071
5|130,802|135,228|122,700|123,484|140,196
AVG.|142,383|137,947|121,627|122,056|145,792

## 心得

看到數據的直覺：是不是跑錯了XD 怎麼加密方法跑得比未加密來得快，不過馬上想到測試的目標是整個傳送的過程不是只有加密行為，加密後如果資料內容壓縮比較大就可以有效降低傳輸的時間，也就符合實際數據的呈現結果

但這次我測試的 message payload 是大資料，如果資料內容換作是小資料，結果說不定會天差地遠

以平均來看 `Snappy` 與 `Lz4` 表現差不多，倒是 `Zstd` 反而跑出不同於其他壓縮方式的結果 (耗時大於完全不壓縮)，在實際的選擇上建議使用真實案例的 message payload 大小來跑跑看數據比較準確

相較於 producer 在發送效率上的差距， consumer 在接收 message 有沒有壓縮的速度差距非常懸殊呀，詳細內容可以參考 [Kafka Consumer 處理速度緩慢，設定調整紀實](https://blog.yowko.com/dotnet-kafka-consumer-setting)

## 參考資訊

1. [Enum CompressionType](https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.CompressionType.html)
2. [Kafka 2.1.0壓縮算法性能測試](https://www.cnblogs.com/huxi2b/p/10330607.html)
3. [Kafka Consumer 處理速度緩慢，設定調整紀實](https://blog.yowko.com/dotnet-kafka-consumer-setting)