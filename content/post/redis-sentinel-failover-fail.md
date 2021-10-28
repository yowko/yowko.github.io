---
title: "Redis 多個 sentinel 無法執行 failover？！"
date: 2017-04-08T20:29:00+08:00
lastmod: 2021-10-28T20:29:51+08:00
draft: false
tags: ["Debug","Redis"]
slug: "redis-sentinel-failover-fail"
aliases:
    - /2017/04/redis-sentinel-failover-fail.html
---
## Redis 多個 sentinel 無法執行 failover？!

這是在為公司架設 Redis sentinel 時遇到的問題：依官方設定至少設定三個 sentinel instance 並設定 quorum 為 `2`，設定上非常順利，但進行 HA 演練時發現 sentinel 沒有正常作用，並沒有如預期地進行 configuration rewrite，看 log 發現各個 sentinel 皆有正確在 master shutdown 後將 master 標示為 "主觀下線"(sdown) 但一直未被標示為 "客觀下線"(odown) 所以 sentinel 無法執行 failover

## 問題主因

照慣例 debug 先說結果：`protected-mode` 造成的

## sentinel 數量建議

官方有明確建議至少三個 instance

![1atleast3](https://cloud.githubusercontent.com/assets/3851540/24828863/52fc5e04-1c99-11e7-9428-26c125e231e8.png)

## 原本 sentinel 設定

```config
port 26379
sentinel monitor master 192.168.31.102 6379 2
sentinel down-after-milliseconds master 3000
sentinel failover-timeout master 18000
sentinel parallel-syncs master 1
sentinel auth-pass master yowkopassword
```

* 仔細檢視 log，可以發現，sentinel 一開始啟動時，其他 sentinel 就被標視為 sdown 了
  * 訊息內容

    ```log
    [19280] 08 Apr 18:37:24.997 # +sdown sentinel a095bbb8638b6062ffae2c0d817fa03bb1260ad3 192.168.31.247 26379 @ master 192.168.31.102 6379
    ```

  * 訊息截圖

    ![2sentineldown](https://cloud.githubusercontent.com/assets/3851540/24828862/52fbedfc-1c99-11e7-98de-9c2d93b93a49.png)

* failover 失敗
  * 訊息內容

    ```log
    [17376] 08 Apr 18:35:17.740 # +try-failover master master 192.168.31.102 6379                                                         
    [17376] 08 Apr 18:35:17.745 # +vote-for-leader 2ae78df0c2add24032cb8f4bb8168ede17848ed3 3                                             
    [17376] 08 Apr 18:35:28.248 # -failover-abort-not-elected master master 192.168.31.102 6379                                           
    [17376] 08 Apr 18:35:28.303 # Next failover delay: I will not start a failover before Sat Apr 08 18:35:54 2017
    ```

  * 訊息截圖

    ![5failovefail](https://cloud.githubusercontent.com/assets/3851540/24828866/53039ee4-1c99-11e7-99d1-0e79436126b9.png)

## 如何修正

原本推斷是 sentinel 推選 leader 失敗造成無法決定出 sentinel leader 來進行 failover，而這個問題的解決方法只要在 sentinel conf 加上 `can-failover` 表示該 sentinel 可以成為 leader 就行了，但測試後發現 `can-failover` 這個設定已被移除， redis 已經不再使用這個設定

經過反覆測試後才診斷發現是 `protected-mode` 造成與其他 sentinel 之間的溝通法無法正確進行

* 解決方式有二個(擇一即可)：
    1. 關閉 `protected-mode`

        > `protected-mode no`

    2. 加上 bind

        > `bind 127.0.0.1 192.168.31.102`

  * 不要只 bind 127.0.0.1  這樣其他 sentinel 還是連不到

* 加上設定並重新啟動後就原本 sentinel sdown 的訊息就不見了

    ![3sentinelOK](https://cloud.githubusercontent.com/assets/3851540/24828864/52fcf076-1c99-11e7-901a-f4c6e458e2df.png)

## 調整後設定

```config
port 26379
bind 127.0.0.1 192.168.31.102
sentinel monitor master 192.168.31.102 6379 2
sentinel down-after-milliseconds master 3000
sentinel failover-timeout master 18000
sentinel parallel-syncs master 1
sentinel auth-pass master yowkopassword
```

## failover 成功

![4failoversucc](https://cloud.githubusercontent.com/assets/3851540/24828865/52fd8608-1c99-11e7-94dc-274393cec7fc.png)

## 參考資料

1. [Redis Sentinel Documentation](https://redis.io/topics/sentinel)
2. [Redis開發運維實踐指南](https://gnuhpc.gitbooks.io/redis-all-about/HAClusterArchPractice/ms/hatest-quorum.html)
