---
title: "批次刪除 Redis 特定 Key"
date: 2020-06-06T21:30:00+08:00
lastmod: 2020-06-06T21:30:31+08:00
draft: false
tags: ["Redis"]
slug: "redis-batch-delete-by-pattern"
---

## 批次刪除 Redis 特定 Key

將 Redis 當做 Cache Server 是滿普遍的做法，修改 Cache 內容格式也是常見的，不過程式部署時連帶清空 redis 可能就不是預設 sop 了

最近專案就遇到類似問題：

1. Redis 預設不清資料
2. 程式在 redis key 存在情況下會直接使用，造成 redis value 內容不符預期
3. Redis 同時存在其他不同 module 資料，只更新某一個 moudle 程式，不能 flush 整個 Redis

雖然這個需求不是常態性存在，執行頻率也不高，但我每次遇到也不免要回憶幾分鐘才組得出正確的指令，重點是每次執行指令都擔心下錯，所以趁著空檔紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker image

    - redis 6.0.4

## 刪除所有 Redis Key 語法

> 適用於單一 Redis database or 單一 Redis instance 只服務某個 module (key 全刪不會影響到其他 module)

1. 使用 `FLUSHDB [ASYNC]`

    > 僅刪除當前的 redis database 所有 key; `ASYNC` 是 Redis 4 之後版本才加上的

    - 語法

        ```bash
        redis-cli [-a {password}] [-h {redis ip}] [-p {redis port}] [-n {db}] FLUSHDB
        ```

    - 範例

        ```bash
        redis-cli -a pass.123 -h 127.0.0.1 -p 6379 -n 2 FLUSHDB
        ```

2. 使用 `FLUSHALL [ASYNC]`

    > 僅刪除所有 redis database 所有 key; `ASYNC` 是 Redis 4 之後版本才加上的

    - 語法

        ```bash
        redis-cli [-a {password}] [-h {redis ip}] [-p {redis port}] FLUSHDB
        ```

    - 範例

        ```bash
        redis-cli -a pass.123 -h 127.0.0.1 -p 6379 FLUSHDB
        ```

## 刪除特定 Redis Key 語法

> 適用 Redis key 有特定 pattern 且不想刪除 Redis 所有 key 的情境 (全刪的時間複雜度是 O(N) - N 是 key 數量)

1. 列出需要刪除的 key

    - 沒有覆寫 `scan` 指令
        - 語法

            ```bash
            redis-cli [-a {password}] [-h {redis ip}] [-p {redis port}] [-n {db}] --scan --pattern {key pattern}
            ```

        - 範例

            ```bash
            redis-cli -a pass.123 -h 127.0.0.1 -p 6379 -n 0 --scan --pattern yowko:*
            ```

    - 覆寫 `scan` 指令

        > redis config 中如果有 `rename-command SCAN` 指令，就無法透過 redis-cli 直接執行 scan，需改用 `keys`

        - 語法

            > 這邊的 `{keys command}` 視有沒有 `rename-command KEYS` 換成實際的 `keys` 指令

            ```bash
            redis-cli [-a {password}] [-h {redis ip}] [-p {redis port}] [-n {db}] {keys command} "{key pattern}"
            ```

        - 範例

            ```bash
            redis-cli -a pass.123 -h 127.0.0.1 -p 6379 -n 0 yowko_keys "yowko:*"
            ```

        - 覆寫 scan 後使用 scan 的錯誤

            ```txt
            Unrecognized option or bad number of args for: '--yowko_scan'
            ```

            ![1error](https://user-images.githubusercontent.com/3851540/83948334-3a2afd00-a84f-11ea-9158-906bc5e26698.jpg)

2. 執行刪除動作

    - Redis 4 之前版本：使用 `del`
        - 語法

            ```bash
            redis-cli [-a {password}] [-h {redis ip}] [-p {redis port}] [-n {db}] del [KEYS]
            ```

        - 範例

            ```bash
            redis-cli -a pass.123 -h 127.0.0.1 -p 6379 -n 0 del yowko:a yowko:b
            ```

    - Redis 4 之後版本：使用 `unlink` (可以背景刪除)

        - 語法

            ```bash
            redis-cli [-a {password}] [-h {redis ip}] [-p {redis port}] [-n {db}] unlink [KEYS]
            ```

        - 範例

            ```bash
            redis-cli -a pass.123 -h 127.0.0.1 -p 6379 -n 0 unlink yowko:a yowko:b
            ```

3. 整合範例

    > 這個就依據上面的語法，搭配適當的環境設定來組裝指令，我先寫好來方便自己抄XD

    ```bash
    redis-cli -a pass.123 -h 127.0.0.1 -p 6379 yowko_keys "yowko:*" | xargs redis-cli -a pass.123 -h 127.0.0.1 -p 6379 unlink
    ```

## 心得

大致上都是熟悉的語法，但是踩到 `rename-command scan` 在 redis-cli 中就無法使用 `scan` 指令有些錯愕，明明 `keys` 能用呀，不過我沒有特別花時間去探討原因  先記下來備忘，要用時有地方參考就好

## 參考資訊

1. [How to Delete Keys Matching a Pattern in Redis](https://rdbtools.com/blog/redis-delete-keys-matching-pattern-using-scan/)
2. [UNLINK](https://redis.io/commands/unlink)
3. [DEL](https://redis.io/commands/del)
4. [FLUSHDB](https://redis.io/commands/flushdb)
5. [FLUSHALL](https://redis.io/commands/flushall)
