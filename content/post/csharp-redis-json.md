---
title: "使用 C# 存取 redis JSON"
date: 2024-07-22T00:30:00+08:00
lastmod: 2024-07-22T00:30:31+08:00
draft: false
tags: ["csharp","redis"]
slug: "csharp-redis-json"
---

## 使用 C# 存取 redis JSON

雖然一直以來持續都在使用 redis，只是用途一直是當做 cache，有訂閱著 redis 的更新消息，但除了 security patch 之外卻沒有額外關注，最近查指令時發現 redis 官網有不少異動，其中一個就是 RedisJSON 的支援，過去剛使用 redis 時，json (這類額外的擴充) 都是由 RedisLabs 提供，所以我個人一直不敢直接使用，擔心日後成為孤兒，現在看到官網直接列入，就可以來試試了

![1redisjson](https://github.com/user-attachments/assets/d62a3199-2000-4db5-9eba-921ca269b3cf)

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack 1.6.3 (17138)
- .NET SDK 8.0.101
- JetBrains Rider 2024.1.4
- NuGet packages
    - Redis.OM 0.7.1
    - NRedisStack 0.12.0
- docker image

    - redis/redis-stack-server:7.2.0-v11

        > 使用 `pass.123` 做為 redis password

        ```bash
        docker run -d --name redis-stack-server -p 6379:6379 -e REDIS_ARGS="--requirepass pass.123" redis/redis-stack-server:7.2.0-v11
        ```

## 使用方式

- Redis.OM

    > 適用於 redis 與 .NET 間的 object mapping，提供以下功能
    >
    >1. 聲明式 object mapping
    >2. 聲明式二級索引
    >3. 查詢 redis 的 fluent api
    >4. redis 聚合操作的 fluent api

    1. 建立 model

        > 1. 使用 Document attribute 標記 class，並指定 StorageType 為 StorageType.Json
        > 2. [GitHub:redis/redis-om-dotnet](https://github.com/redis/redis-om-dotnet) 上有提到要定義 index，雖然沒有定義也可以成功新增，但後續查詢時會出現錯誤
        > 3. 物件的 id 是 unique 也是 redis primary index key 的一部份，Redis.OM 原生支援的 id 類型是 ulid (想多了解 ulid 可以參考之前筆記 [.NET 中的 UUID(GUID) 與 ULID](/dotnet-uuid-guid-ulid/))：預設使用 `{model 類別名稱}:{ULID}` 做為 redis key；`model 類別名稱` 可以透過 DocumentAttribute 的 Prefixes 來修改 (只能設定為非空字串的值)；`ULID` 則可以使用 RedisIdField attribute 來指定特定 property 來當作 id

        - 全未指定

            > key 為 `User:01J3CKPVQDZRRDJDGSM9KQF8Z8` (ULID 僅為範例)

            {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "model.cs">}}

        - 全指定 (指定 prefix 可能造成查詢異常)

            > key 為 `userprefix:yowko`

            {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "modelwithkey.cs">}}

        - 指定 prefix 為 `""` (效果等同未指定 prefix)

            > key 為 `User:yowko`

            {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "modelemptyprefix.cs">}}

        - 未指定 index (查詢時會出現錯誤)

            {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "noindex.txt">}}

            ![2noindex](https://github.com/user-attachments/assets/ea3eda97-d4b7-4954-a83a-d4bf04fdf236)

    2. 新增資料

        {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "insert.cs">}}

    3. 查詢資料

        {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "query.cs">}}

        - 查詢時 prefix 失效：只能查詢 `{model 類別名稱}:*` 的內容

          - 有 CreateIndex
            {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "emptyresult2.cs">}}

          - 無 CreateIndex
            {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "emptyresult.cs">}}

    4. 完整程式碼

        {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "RedisOM.cs">}}

- NRedisStack

    基於 StackExchange.Redis 建立，可以為 C# 提供對 Redis Stack 指令的支援 (不止於 JSON，也包含 Search、TimeSeries、Bloom Filter、Cuckoo Filter、T-Digest、Count-min Sketch 和 Top-K 都有對應的指令)

    1. 新增資料

        {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "NRedisStackInsert.cs" >}}

    2. 建立 index 與查詢資料

        {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "indexquery.cs" >}}

    3. 完整程式碼

        {{<gist yowko e8f2ee409f0e5fbafaab7d0cb9ead167 "NRedisStack.cs" >}}

## 心得

- 其他 RedisJSON module 的使用方式，可以參考 [Enable Redis JSON](https://redis.io/docs/latest/develop/data-types/json/#enable-redis-json)
- redis image 的選擇：

    - redis：包含 String、Hash、List、Set、Sorted Set、Bitmaps、HyperLogLog、Geospatial Indexes、Streams 的 key-value pair database
    - redis-stack-server：redis 加上 RedisJSON、RedisTimeSeries、RedisBloom、RedisGraph
    - redis-stack：redis-stack-server + RedisInsight

- 使用 redis-cli 存取 json

    - set

        - 語法

            ```bash
            JSON.SET {key} $ '{json}'
            ```

        - 範例

            ```bash
            JSON.SET yowko $ '{"Username":"yowko","Email":"yowko@yowko.com","Rank":99,"Birthday":428256000000}'
            ```

        ![3cliset](https://github.com/user-attachments/assets/fb7916d4-6833-48b6-a546-a4a2a660fafa)

    - get

        - 語法

            ```bash
            JSON.GET {key}
            ```

        - 範例

            ```bash
            JSON.GET yowko
            ```

        ![4cliget](https://github.com/user-attachments/assets/4513f63a-44d8-4d72-b562-dde7f17f558f)

- 其他錯誤：我只遇到一次 Redis.OM 的錯誤，但是我無法重現，所以無法確認原因

    ![4redisomerror](https://github.com/user-attachments/assets/8738a060-9bf2-4d49-9869-48dbbd7991b5)

- 兩者比較

    我個人覺得如果只是單純的存取 JSON，Redis.OM 比較簡單，但是如果需要較彈性的設定，NRedisStack 會比較適合

    - Redis.OM 在 model mapping 上較為方便，index 也不需自行處理，但建不出沒有 prefix 的 key
    - NRedisStack 較為彈性，但需要自行處理 index，語法上也需要同時 JSON 跟 Search，除此之外也包含其他 Redis module 的支援

        > 不確定是不是我誤會，但依照 [GitHub:redis/NRedisStack](https://github.com/redis/NRedisStack) readme 的說明，使用 `NRedisStack` 時，無法順利查詢資料，後來在 [GitHub:examples folder](https://github.com/redis/NRedisStack/blob/master/Examples/BasicQueryOperations.md) 找到成功範例

## 參考資訊

1. [.NET 中的 UUID(GUID) 與 ULID](/dotnet-uuid-guid-ulid/)
2. [Understand Redis data types](https://redis.io/docs/latest/develop/data-types/)
3. [Run Redis Stack on Docker](https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/docker/)
4. [Enable Redis JSON](https://redis.io/docs/latest/develop/data-types/json/#enable-redis-json)
5. [GitHub:redis/redis-om-dotnet](https://github.com/redis/redis-om-dotnet)
6. [GitHub:redis/NRedisStack](https://github.com/redis/NRedisStack)
7. [GitHub:examples folder](https://github.com/redis/NRedisStack/blob/master/Examples/BasicQueryOperations.md)
