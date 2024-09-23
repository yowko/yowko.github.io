---
title: "使用 C# 與 redis 避免重複資料"
date: 2024-07-30T00:30:00+08:00
lastmod: 2024-09-23T00:30:31+08:00
draft: false
tags: ["csharp","redis"]
slug: "csharp-redis-avoid-duplicate"
---

## 使用 C# 與 redis 避免重複資料

過去在面對可能重複的資料時，常見的做法是透過關聯式資料庫的 unique key 來避免重複資料的產生，但資料庫的資源相對稀缺、加上資料庫的 IO 操作相對耗時，所以在高頻系統的大量操作下，可能會造成資料庫的效能問題，所以近幾年類似的需求，就改透過 redis sets 唯一字串(成員)的特性來達成，前陣子瀏覽 redis stack 的文件時，瞄到 Bloom filter 剛好想起之前看過的一篇相關文章 [Redis & .NET for Data Deduplication](https://medium.com/codenx/redis-net-for-data-deduplication-c47af1f004b1)，所以就來試試不同做法

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack 1.6.3 (17138)
- .NET SDK 8.0.101
- JetBrains Rider 2024.1.4
- NuGet packages
    - StackExchange.Redis 2.8.16
    - NRedisStack 0.12.0
- docker image

    - redis/redis-stack:7.4.0-v0

        > 使用 `pass.123` 做為 redis password；redis-stack 比 redis-stack-server 多了 RedisInsight (可以用 ui 來檢視 redis 資料)

        ```bash
        docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 -e REDIS_ARGS="--requirepass pass.123" redis/redis-stack:7.4.0-v0
        ```

## 使用方式

1. Redis Sets

    {{<gist yowko eafe9b2ba86d2fd3f91f6617176e7e08 "redissets.cs">}}

2. Redis Bloom filter

    {{<gist yowko eafe9b2ba86d2fd3f91f6617176e7e08 "bloomfilter.cs">}}

## 心得

1. Redis sets

    - 優點：
        - 保證唯一性
        - 簡單易用
    - 缺點：
        - 儲存所有資料，佔用量大
        - 速度會因資料量增加而下降

2. Redis Bloom filter

    - 優點：
        - 僅儲存 hash 值，佔用量小
        - 速度不會因資料量增加而下降
        - 可以設定過濾器大小、hash 函數數量
    - 缺點：
        - 有一定的碰撞風險

3. 針對兩者的 memory 耗用量

    - 建立模擬資料 100、10000、100000 筆，分別測試 memory 耗用量

        {{<gist yowko eafe9b2ba86d2fd3f91f6617176e7e08 "memory.cs">}}

    - 實際耗用量

        ![1memory](https://github.com/user-attachments/assets/711c8304-9bd2-4629-b473-3edda56ef045)

完整程式碼請參考 [GitHub:yowko/csharp-redis-avoid-duplicate](https://github.com/yowko/redis-deduplicate)

## 參考資料

1. [Redis & .NET for Data Deduplication](https://medium.com/codenx/redis-net-for-data-deduplication-c47af1f004b1)
2. [Redis sets](https://redis.io/docs/latest/develop/data-types/sets/)
3. [Bloom filter](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/)
4. [GitHub:yowko/csharp-redis-avoid-duplicate](https://github.com/yowko/redis-deduplicate)
