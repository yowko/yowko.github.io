---
title: "使用 C# 取得 Redis 複雜型別 table 資料"
date: 2024-04-21T00:30:00+08:00
lastmod: 2024-04-21T00:30:31+08:00
draft: false
tags: ["csharp","redis"]
slug: "csharp-redis-table"
---

## 使用 C# 取得 Redis 複雜型別 table 資料

之前筆記 [使用 C# 取得 Redis key 的記憶體用量](/csharp-redis-key-memory-usage) 提到如何使用 C# 取得 Redis key 的記憶體用量，但是把所有 redis key 全部拉回 C# 做處理，可以想見執行速度一定會有不少提升空間，至少網路傳輸的時間就是不必要的浪費，所以打算在 lua 中將 key 以群組的方式來匯總資料，以加快資料取得的速度，於是便在 lua 中透過 table 的方式來儲存資料，可是在回傳至 C# 卻一直收到空資料，這時想起過去也有篇筆記 [C# 使用 Lua 取得 Redis 自訂複雜型別](/csharp-lua-redis-custom-type/) 紀錄過做法，只是當時沒有留下完整範例，造成無法重現當時成功的場景，因此利用這次機會紀錄一下

## 基本環境說明

- macOS Sonoma 14.3.1 (Apple M2 Pro)
- .NET SDK 8.0.101
- OrbStack 1.5.0 (16835)
- JetBrains Rider 2024.1
- docker image

    - redis:7.2.4

- Nuget package

    - StackExchange.Redis 2.7.33
    - MessagePack 2.5.140

- redis cluster docker compose

    > 啟動：`ip=$(ipconfig getifaddr en0) docker-compose up -d` (`en0` 是使用中的網路介面名稱)

    {{< gist yowko ccb4654756c89dd0c602ee2ade5a53ea "docker-compose.yml" >}}

- 測試資料

    > 加上 hash tag，將 key 集中在某些 slot 上，以利後續測試

    ```bash
    set {yowko}test:1 13b2eb75df52465d0896e2381481aec35c81a2cb
    ```

    ```bash
    set {yowko}test:2 7f058a43ab77bb3cd48e446feeadbd0e
    ```

    ```bash
    set {yowko}develop:3 ccb4654756c89dd0c602ee2ade5a53ea
    ```

## Debug Lua

這一步我就遇到不小的困難，因為 debug lua 需要使用到 `--eval` 來載入 lua，但團隊的 redis 有設定 rename 部份 command，造成無法使用 `--eval` 來 debug lua，這個我沒有找到解決辦法，所以我只能 local 建立一個沒有 rename command 的 redis instance 來 debug

1. redis.lua

    {{< gist yowko  7f058a43ab77bb3cd48e446feeadbd0e "redis.lua" >}}

2. 使用 redis-cli 執行 debug

    - 語法

        ```bash
        redis-cli --ldb --eval {lua file} KEYS[] , ARGV[]
        ```

    - 範例：參數沒有指定名稱，且有兩個參數

        ```bash
        redis-cli -p 7002 --ldb --eval redis.lua  , 12231 12231
        ```

3. 實際 debug

    - `step` 逐步執行
    - `redis.debug()` 可以輸出變數的值
    - `continue` 執行至這個 breakpoint 或是 結束
    - `help` 列出所有指令

        ![1debug](https://github.com/yowko/picsbed/assets/3851540/a61fd4b1-0e78-4fc7-ad00-220b6eb3e7b0)

4. lua table 有 key value，則會回傳 empty array

    ![2emptryresult](https://github.com/yowko/picsbed/assets/3851540/f9614848-ac41-47b3-8626-f96c4d64ed7b)

## C# 存取 Lua 回傳的 table 資料

為了避開 `lua table 有 key value，則會回傳 empty array` 的問題，可以將 lua table 做 serilize，這樣就可以在 C# 中正確取得資料

1. 使用 json

    {{< gist yowko 7f058a43ab77bb3cd48e446feeadbd0e "Program_Json.cs" >}}

2. 使用 messagePack

    {{< gist yowko 7f058a43ab77bb3cd48e446feeadbd0e "Program_MsgPack.cs" >}}

## 心得

這篇筆記比較偏向之前筆記 [C# 使用 Lua 取得 Redis 自訂複雜型別](/csharp-lua-redis-custom-type/) 的補充，主要是為了將環境設定完整紀錄下來，避免日後找不到相關資料，另外也補上 lua debug 的過程

完整程式碼請參考：[GitHub:yowko/RedisGroupMemoryUsage](https://github.com/yowko/RedisGroupMemoryUsage)

## 參考資料

1. [使用 C# 取得 Redis key 的記憶體用量](/csharp-redis-key-memory-usage)
2. [C# 使用 Lua 取得 Redis 自訂複雜型別](/csharp-lua-redis-custom-type/)
3. [Redis - Lua tables as return values - why is this not working](https://stackoverflow.com/a/37222741)
4. [Redis 如何调试Lua 脚本](https://zhuanlan.zhihu.com/p/353860023)
5. [redis中lua脚本的简单使用](https://www.cnblogs.com/huan1993/p/15472920.html)
6. [GitHub:yowko/RedisGroupMemoryUsage](https://github.com/yowko/RedisGroupMemoryUsage)
