---
title: "Kafka Consumer 處理速度緩慢，設定調整紀實"
date: 2019-06-01T19:30:00+08:00
lastmod: 2021-11-03T19:30:31+08:00
draft: false
tags: ["Kafka","dotnet core"]
slug: "dotnet-kafka-consumer-setting"
---

## Kafka Consumer 處理速度緩慢，設定調整紀實

最近專案用了 Kafka 做為中介的 message queue，近期已經陸陸續續開始準備上線的前置作業，其中一項就是壓測數據，不過 Kafka 的 consumer 在核心 component 間處理大量資料的表現非常不好，完全沒有展現出該有的效能水準，於是就得來進行效能調校，但筆記的標題實在不敢提到效能調校，怕被同事幾位大大恥笑 XD, 就簡單紀錄一下我調整的參數值與效能數據結果

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)
3. Docker version 18.09.2, build 6247962
4. NuGet package

    - Confluent.Kafka 1.0.1

5. Kafak 環境建置使用 [wurstmeister/kafka-docker](https://github.com/wurstmeister/kafka-docker)
6. 情境說明

    > 透過 docker-compose 啟動 kafka 以及 zookeeper，接著使用 console application 傳送 10萬則內容為 20267 字的 message，完成後再啟動 consumer 來接收 message 並計算總耗時

## 實際執行數據

> 以下數據單位皆為 `毫秒(Milliseconds)`

- 只調整 Consumer 設定
    1. 全部預設值

        測試序|實際結果
        :---|:---
        1|27487.108
        2|26793.98
        3|26173.051
        avg|26818.04633

    2. 調整 Consumer fetch.max.bytes 為設定值 (52428800) 兩倍

        測試序|實際結果
        :---|:---
        1|26800.383
        2|25892.785
        3|28924.617
        AVG.|27205.92833

    3. 調整 Consumer max.partition.fetch.bytes 為設定值 (1048576) 兩倍

        測試序|實際結果
        :---|:---
        1|23810.762
        2|23872.324
        3|25626.072
        AVG.|24436.386

    4. 同時調整 fetch.max.bytes 為設定值 (52428800) 兩倍 與 max.partition.fetch.bytes 為設定值 (1048576) 兩倍

        測試序|實際結果
        :---|:---
        1|25317.314
        2|24860.095
        3|25113.587
        AVG.|25096.99867

- 調整 topic
    1. 調整 TOPIC max.message.bytes 為預設值 (1000012) 兩倍

        測試序|實際結果
        :---|:---
        1|26565.841
        2|25682.327
        3|25744.871
        AVG.|25997.67967

    2. 調整 TOPIC max.message.bytes 為預設值 (1000012) 兩倍搭 Consumer fetch.max.bytes 為設定值 (52428800) 兩倍

        測試序|實際結果
        :---|:---
        1|25268.01
        2|25987.696
        3|26499.988
        AVG.|25918.56467

    3. 調整 TOPIC max.message.bytes 為預設值 (1000012) 兩倍搭 max.partition.fetch.bytes 為設定值 (1048576) 兩倍

        測試序|實際結果
        :---|:---
        1|24769.288
        2|25227.366
        3|25406.308
        AVG.|25134.32067

    4. 調整 TOPIC max.message.bytes 為預設值 (1000012) 兩倍 搭 max.partition.fetch.bytes 為設定值 (1048576) 兩倍搭 Consumer fetch.max.bytes 為設定值 (52428800) 兩倍

        測試序|實際結果
        :---|:---
        1|24641.013
        2|24487.387
        3|24973.881
        AVG.|24700.76033

- 其他嘗試

    1. 透過 per tool

        測試序|實際結果
        :---|:---
        1|27108
        2|29287
        3|28161
        AVG.|28185.33333

    2. perf tool thread 2

        測試序|實際結果
        :---|:---
        1|27998
        2|29655
        3|27693
        AVG.|28448.66667

    3. 2 broker

        測試序|實際結果
        :---|:---
        1|30833.284
        2|27955.218
        3|28015.344
        AVG.|28934.61533

    4. message 壓縮

        壓縮類型|實際結果
        :---|:---
        None|27445.124
        Gzip|4449.036
        Snappy|3043.064
        Lz4|2675.844
        Zstd|3885.105

## 心得

調整 `max.partition.fetch.bytes` 看似有效，只是效果不明顯(約 8.8%)，而調整 `fetch.max.bytes` 後速度則沒有顯著改善，加上 topic 的調整也是如此，速度的提升都沒有超過 10%

於是我想到會不會根本不是 c# 程式的問題， 而是 kafka 本身在所設定的情境下就很難有較佳的處理速度，於是改用 kafka 內建的 perf 工具測試，果然證實了我的想法，速度與之前數據差不多

最後嘗試透過壓縮 message payload 的方式來測試，果然速度上得到大量的提昇，上述壓縮測試僅就 consumer 端，如果需要一併考量 producer 的執行效率可以參考 [Kafka Producer 不同壓縮方式對發送速度的影響](/dotnet-kafka-producer-compresstype) 再選擇壓縮方式

## 參考資訊

1. [Kafka Producer 不同壓縮方式對發送速度的影響](/dotnet-kafka-producer-compresstype)
