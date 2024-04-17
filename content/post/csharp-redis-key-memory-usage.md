---
title: "使用 C# 取得 Redis key 的記憶體用量"
date: 2024-04-17T00:30:00+08:00
lastmod: 2024-04-17T00:30:31+08:00
draft: false
tags: ["csharp","redis"]
slug: "csharp-redis-key-memory-usage"
---

## 使用 C# 取得 Redis key 的記憶體用量

團隊某個 redis cluster 中，有一組 replication (master-replica) 的 memory 用量特別高，推測是 key 的 hash tag 不夠分散，造成 key 都集中在某些 slot 上，進而使得一組 cluster replication (master-replica) 的 memory 用量特別高。

為了明確地定位問題來源，打算透過 C# 搭配 lua 來取得 redis key 的記憶體用量。

曾經嘗試找過其他工具，不過因為團隊的 redis 有設定 rename 部份 command，造成有些工具無法正常運作，所以決定自己寫程式來解決問題，順便紀錄一下過程跟環境設定，避免日後找不到相關資料。

## 基本環境說明

- macOS Sonoma 14.3.1 (Apple M2 Pro)
- .NET SDK 8.0.101
- OrbStack 1.5.0 (16835)
- JetBrains Rider 2024.1
- docker image

    - redis:7.2.4

- Nuget package

    - StackExchange.Redis 2.7.33

- redis cluster docker compose

    > 啟動：`ip=$(ipconfig getifaddr en0) docker-compose up -d` (`en0` 是使用中的網路介面名稱)

    {{< gist yowko ccb4654756c89dd0c602ee2ade5a53ea "docker-compose.yml" >}}

- 測試資料

    > 加上 hash tag，將 key 集中在某些 slot 上，以利後續測試

    ```bash
    set {yowko}test 13b2eb75df52465d0896e2381481aec35c81a2cb
    ```

## 使用方式

{{< gist yowko ccb4654756c89dd0c602ee2ade5a53ea "Program.cs" >}}

## 心得

我紀錄一下我嘗試過的做法，讓大家在決定怎麼取得 redis key 的記憶體用量時，提供一些想法

1. 使用 bash ：[picserve/redis_key_sizes.sh](https://gist.github.com/epicserve/5699837)

    - 優點：bash script 一目了然，且不需要額外安裝套件
    - 缺點：bash 的語法不太好閱讀，且不容易擴充，速度上不是很快

2. 使用 [obukhov/redis-inventory](https://github.com/obukhov/redis-inventory)

    - 優點：輸出漂亮，還有階層顯示，甚至可以有 sunset chart
    - 缺點： rename-command 要自行修改 source code，我改完後跑不出結果，但也顯示錯誤訊息，不知道問題在哪

3. 使用 C# 透過 StackExchange.Redis 來取得 key 的記憶體用量

    - 優點：直覺，不用同時 debug c# 跟 lua
    - 缺點：太慢了，每個 key 都要 access redis 一次

完整程式碼，請參考：[GitHub:yowko/redis-key-memory-usage](https://github.com/yowko/redis-key-memory-usage)

## 參考資料

1. [Redis｜用 Docker 架設 Redis Cluster](https://blog.bimap.com.tw/2021/06/10/dockerize-redis-cluster-tutorial)
2. [picserve/redis_key_sizes.sh](https://gist.github.com/epicserve/5699837)
3. [GitHub:obukhov/redis-inventory](https://github.com/obukhov/redis-inventory)
4. [GitHub:yowko/redis-key-memory-usage](https://github.com/yowko/redis-key-memory-usage)
