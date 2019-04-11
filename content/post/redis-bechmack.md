---
title: "Redis bechmack 用法與參數說明"
date: 2017-03-22T02:43:34+08:00
lastmod: 2018-09-14T00:42:34+08:00
draft: false
tags: ["Redis"]
slug: "redis-bechmack"
aliases:
    - /2017/03/redis-bechmack.html
---
# Redis bechmack 用法與參數說明
透過 redis-benchmark 這個 redis 內建工具可以幫助我們瞭解 redis 環境的執行效能，測試過程中隨手紀錄一下指令用法

## 如何使用

```
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]
```

## 參數說明

參數|預設值|說明
:---|:---|---
 -h {hostname}|127.0.0.1| redis server 的主機名稱或 ip
 -p {port|6379|redis server 的 port
 -s {socket}    |-|redis server socket(會蓋掉 host 和 port 設定)
 -a {password}| -|redis 的連線密碼    Password for Redis Auth
 -c {clients} |50 |平行連線數
 -n {requests} |100000|總 request 數量
 -d {size}  |2| 指定 SET/GET 資料的 bytes 數
 -dbnum {db}  |0|指定使用特定 db 編號 (windows 環境似乎不支援)
 -k {boolean}  |1   |  1=持續連線 0=重新連線
 -r {keyspacelen}|__rand_int__<br/>未指定就固定用這個字串|SET/GET/INCR 指令使用動態的 key (key:__rand_int__)來測試 , SADD 使用動態的 value (element:__rand_int__)來測試，每次執行時會隨機從 0 到 keyspacelen-1 間產生一個 12 位數的數字來取代 `__rand_int__`
 -P {numreq}|1<br/>(停用 pipeline)|指定 Pipeline 數量來發生 request
 -q     |-|只顯示 query/sec 結果，其他詳細資訊不顯示
 --csv   |-|將結果以 CSV 格式輸出
 -l       |-|持續執行測試
 -t {tests}|-|僅執行指定的指令測試，多個指令以 `,` 分隔
 -I|-|建立 idle 連數，數量由 `-c` 決定


# 參考資訊
1. [How fast is Redis?](https://redis.io/topics/benchmarks)