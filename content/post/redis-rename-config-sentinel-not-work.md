---
title: "Redis Rename Config 後 Sentinel 無法正確執行 Failover"
date: 2020-02-16T21:30:00+08:00
lastmod: 2020-02-16T21:30:31+08:00
draft: false
tags: ["Linux","Redis","Ansible"]
slug: "redis-rename-config-sentinel-not-work"
---

## Redis Rename Config 後 Sentinel 無法正確執行 Failover

有些 Redis command (e.g. `KEYS`,`SAVE`) 在執行時會 block 其他操作，在 Redis 用量小時不明顯，但 Redis 日漸龐大後，執行這些 command 時會讓 Redis 被 block 的時間愈來愈長，最後可能就出現逾時，為了避免這類 command 造成不必要的效能損耗，最好是停用相關 command；除此之外，資安風險相關的 command (e.g. `SHUTDOWN`,`FLUSHALL`,`FLUSHDB`,`CONFIG`)也該被留意

今天就來紀錄一下，前幾天安裝 Redis Replication 時停用 `CONFIG` 時遇到的問題

## 基本環境說明

1. Azure VM B1s (1 vcpu,2GiB memory) X 3
2. CentOS-based 7.7
3. Redis 5.0.7
4. epel-release.noarch 0:7-11
5. ius-release.noarch 0:2-1.el7.ius
6. ansible 2.9.3
7. python 2.7.5

## 問題描述

1. 安裝 Redis Replication (1 master、1 slave、3 sentinel)

    > 使用 [在 CentOS 7 上安裝 Redis Replication (Redis 5)](https://blog.yowko.com/install-redis) or [使用 Ansible 安裝 Redis Replication](https://blog.yowko.com/ansible-install-redis) 安裝，而且停用了 `CONFIG` (`rename-command CONFIG ""`)

2. 測試 failover 時，slave 無法順利成為 master

    ```bash
    redis-cli -a pass.123 -h 10.0.0.12 -p 6379 shutdown
    ```

    > slave 已偵測到 master 狀態為 down，但仍然無法提升為 master

    ![1slave](https://user-images.githubusercontent.com/3851540/74606374-f464f000-510a-11ea-987f-530f98245a1d.png)

3. sentinel 執行狀況

    - sentinel 偵測到 master `odown`

        ```txt
        # +new-epoch 1
        # +vote-for-leader 1297f81c784c9ccc81e8ba2ca983a457960d199d 1
        # +sdown master master_2 10.0.0.12 6379
        # +odown master master_2 10.0.0.12 6379 #quorum 1/1
        # Next failover delay: I will not start a failover before Sun Feb 16 13:56:52 2020
        ```

        ![3sentinel](https://user-images.githubusercontent.com/3851540/74606377-f75fe080-510a-11ea-8db9-2cdeeb4666e5.png)

    - failover 執行 promote slave 失敗

        ```txt
            # +sdown master master_2 10.0.0.12 6379
            # +odown master master_2 10.0.0.12 6379 #quorum 1/1
            # +new-epoch 1
            # +try-failover master master_2 10.0.0.12 6379
            # +sdown master master_2 10.0.0.12 6379
            # +odown master master_2 10.0.0.12 6379 #quorum 1/1
            # +new-epoch 1
            # +try-failover master master_2 10.0.0.12 6379
            # +vote-for-leader 1297f81c784c9ccc81e8ba2ca983a457960d199d 1
            # +vote-for-leader 154b385a9a326b33547f1c28b57c79357e8dddbc 1
            # 154b385a9a326b33547f1c28b57c79357e8dddbc voted for        154b385a9a326b33547f1c28b57c79357e8dddbc 1
            # 1297f81c784c9ccc81e8ba2ca983a457960d199d voted for        1297f81c784c9ccc81e8ba2ca983a457960d199d 1
            # 2f747c6ae71701cd930600ef596690a6b88a5aa6 voted for        1297f81c784c9ccc81e8ba2ca983a457960d199d 1
            # 2f747c6ae71701cd930600ef596690a6b88a5aa6 voted for        1297f81c784c9ccc81e8ba2ca983a457960d199d 1
            # +elected-leader master master_2 10.0.0.12 6379
            # +failover-state-select-slave master master_2 10.0.0.12 6379
            # +selected-slave slave 10.0.0.13:6380 10.0.0.13 6380 @ master_2 10.0.0.12 6379
            * +failover-state-send-slaveof-noone slave 10.0.0.13:6380 10.0.0.13 6380 @ master_2 10.0.       0.12 6379
            * +failover-state-wait-promotion slave 10.0.0.13:6380 10.0.0.13 6380 @ master_2 10.0.0.     12 6379
            # -failover-abort-not-elected master master_2 10.0.0.12 6379
            # Next failover delay: I will not start a failover before Sun Feb 16 13:56:52 2020
            # -failover-abort-slave-timeout master master_2 10.0.0.12 6379
            # Next failover delay: I will not start a failover before Sun Feb 16 13:56:52 2020
        ```

        ![4promotefail](https://user-images.githubusercontent.com/3851540/74606378-f8910d80-510a-11ea-98ee-73e73939e2bf.png)

## 解決方式

> 將 `CONFIG` rename 而不是完全停用，並讓 sentinel 也使用對應的 rename 後 command

- 避免使用 `rename-command CONFIG ""`

    ```conf
    rename-command CONFIG YOWKO_CONFIG
    ```

- sentinel config 中也要 rename

    ```conf
    sentinel rename-command master_{{loop.index}} CONFIG YOWKO_CONFIG
    ```

## 心得

問題發生原因就是因為 sentinel 會伸手進去修改 redis node 的 config 內容，原本以為 sentinel 會用內部指令來執行，結果看來 sentinel 的指令跟 client 是相同的，所以需要在 sentinel config 中將 rename 的指令加上，sentinel 才能正確執行 failover

這才發現雖然以前就知道 sentinel 會修改 master-slave 的 config 內容，但以前安裝時都沒特別 rename config 才沒遇到這問題，自以為熟悉的東西在不同情境下還是可以有不同發現，挺有趣的 ~~

## 參考資訊

1. [在 CentOS 7 上安裝 Redis Replication (Redis 5)](https://blog.yowko.com/install-redis)
2. [使用 Ansible 安裝 Redis Replication](https://blog.yowko.com/ansible-install-redis)
3. [Sentinel and renamed CONFIG command](https://github.com/antirez/redis/issues/2733#issuecomment-495170882)
4. [Rename of SENTINEL command breaks redis-server auto failover to slave](https://github.com/antirez/redis/issues/5092#issuecomment-402231232)
