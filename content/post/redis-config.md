---
title: "Redis 3.2.8 與 Redis-Windows 3.2.1 config 解析 - part 1 (INCLUDES,NETWORK,GENERAL)"
date: 2017-04-01T02:16:00+08:00
lastmod: 2021-10-28T11:08:29+08:00
draft: false
tags: ["Redis"]
slug: "redis-config"
aliases:
    - /2017/04/redis-config.html
---
## Redis 3.2.8 與 Redis-Windows 3.2.1 config 解析 - part 1 (INCLUDES,NETWORK,GENERAL)

前幾天在進行 Redis Server HA 設定時，發現有好幾個設定自己沒有實際使用過，更重要的是還有個設定我之前根本就誤會使用方式了，身為一個期望自己更好的工程師，一定要立馬來好好搞清楚才行，不然鬧了笑話事小，產生 bug 可就嚴重了

## 關於單位(units)

> config 範例一開始指出關於 Redis 使用 memory 容量的單位說明

1. 單位量詞無大小寫之分
2. 單位用法
    * 1k => 1000 bytes
    * 1kb => 1024 bytes
    * 1m => 1000000 bytes
    * 1mb => 1024*1024 bytes
    * 1g => 1000000000 bytes
    * 1gb => 1024*1024*1024 bytes

## 關於引用(INCLUDES)

1. 可以引用一個或多個 config template 來標準化管理 Redis
2. 被引用的檔案需要被設定成可被引用
3. `include` 選項無法被 admin 或是 sentinel 的 `config rewrite` 指令覆寫
4. 將 include 寫在 config 開頭可以避免在執行時期被其他指令覆寫
5. 如果需要覆寫 include，就將指令寫在文件結尾，確保會被當做最後的值
   * 範例：
       * `include .\path\to\local.conf`
       * `include c:\path\to\other.conf`

## 關於網路(NETWORK)

* 關於 bind
    > 如果沒有使用 `bind` 指令來設定， Redis 預設會監聽 server 上的可用網路 IP(我搞錯的設定就是這個，~~我原以為是白名單的概念，僅接受設定者的連線~~)

    1. 可以使用 `bind` 指定監聽一個或多個 ip
    2. 如果 Redis 直接對外，監聽所有可用 IP 是非常危險的
       * 範例
           * 只允許透過 `127.0.0.1` 進行連線

               > `bind 127.0.0.1`

           * 允許透過 192.16.1.100 及 10.0.01 的連線

               > `bind 192.168.1.100 10.0.0.1`

* 關於 protected-mode

    > 為了避免 Redis 直接曝露在網路，預設啟用保護模式。
保護模式啟用下，如果同時滿足以下兩個條件， Redis 只能接受 127.0.0.1 與 ::1 或是 unix socket 的連線：

    1. 沒有使用 `bind` 指令明確指定監聽接口
    2. 沒有使用 `requirepass` 設定密碼
       * 範例
           * `protected-mode yes #啟用保護模式`
           * `protected-mode no #停用保護模式`

* 關於 port
    1. 指定 Redis 接受連線的 port (預設 6379)
    2. 若設為 0，會停用 TCP socket
       * 範例：

           > `port 6379`

* 關於 TCP listen() backlog

    > 這個部份我看不懂，以下僅提供個人理解用

    1. 屬於 linux kernel 的部份，TCP listen() 可以設定 handshake 處理的 queue 大小
    2. 在高用量的環境，建議將 backlog 設大，以避免 client 連線緩慢
    3. 相關資訊可以參考 [Linux TCP socket 開發中 listen backlog 的含義](http://www.jianshu.com/p/fe2228a77429),[listen() 的 backlog 及 TCP 相關參數](http://blog.clanzx.net/2014/05/17/listen-backlog.html)

  * 範例

        > `tcp-backlog 511`

* 關於 Unix socket
  * 用來指定 unix socket 的連線路徑，預設不啟用監聽 unix socket
  * 範例

        > `unixsocket /tmp/redis.sock`

* 關於 timeout
  * 指定秒數關閉閒置的連線，預設為 0 (不啟用)
  * 範例

        > `timeout 0`

* 關於 TCP keepalive
  * 如果值不為 0 則設定 TCP 的 SO_KEEPALIVE 值，主要有兩個好處：
        1. 檢測失效的連線
        2. 避免連接兩端的網路設備有問題而出現正常連線的假象

  * 指定秒數發送 TCP ACK  訊號來進行確認
  * 偵測到關閉連線需耗費兩倍設定值的時間
  * Redis 3.2.1 開始預設值為 300 秒

  * 範例

        > `tcp-keepalive 300`

## 關於一般設定(GENERAL)

* 關於 daemon
  * 預設不以 daemon(背景 service) 執行，以 command 方式啟動時會令 command prompt 進入互動模式
  * 如果啟用 daemon，會將資料寫入 pid file
  * windows - Redis 3.2.1 版不支援這個設定
  * 範例

        `daemonize no`

* 關於 supervised

    > 這個選項我看不懂用途，以下提供個人理解用

  * 透過 `upstart` 和 `systemd` 指令來管理 daemon 用
  * 可以用來確保 Redis 會自動重啟
  * windows - Redis 3.2.1 版不支援這個設定
  * 範例

        > `supervised no`

* 關於 pidfile
  * 用來指定 pid 路徑，會在啟動時寫入(建立檔案)，結束時移除(檔案)
  * 如果不是在 daemon 模式下`且`未指定 pid 路徑就不會建立 pid 檔
  * 在daemon 模式下，需要使用 pid 檔，此時會使用預設路徑 `/var/run/redis_6379.pid`
  * windows - Redis 3.2.1 版不支援這個設定
  * 範例

        > `pidfile /var/run/redis_6379.pid`

* 關於 loglevel
  * 指定 log 級別
    * debug:最大量 log，適用於開發、測試階段
    * verbose:次多 log
    * notice:適量 log，適合 production
    * warning:僅重要、關鍵 log

  * 範例

        > `loglevel notice`

* 關於 logfile
  * 指定 log 位置及檔名
  * 如果是空字串，就會使用 stdout(standard output) - windows 版 3.2.1 則是要指定 `stdout`
  * 使用 daemon 且使用 standard output 時，log 會寫至 `dev/null`
  * 範例

        > `logfile ""`

* 關於 syslog-enabled
  * 將 log 寫至系統 log 中
  * windows 環境下，使用 service 執行，會自動啟用
  * 範例

        > `syslog-enabled no`

* 關於 syslog-ident
  * 設定 系統 log 名稱
  * 範例

        > `syslog-enabled no`

* 關於 syslog-facility
  * 設定 syslog 設備
  * 必需是 USER 或者 LOCAL0 到 LOCAL7
  * windows 版 3.2.1 不支援這個設定
  * 範例

        > `syslog-facility local0`

* 關於 databases
  * 指定 redis instance db 數量
  * 連線時未指定 db id ，預設使用 db 0
  * 範例

        > `databases 16`

## 參考資料

1. [redis/redis.conf](https://github.com/MSOpenTech/redis/blob/3.0/redis.conf)
2. [創建 redis systemd 服務](https://iyaozhen.com/systemd-service-for-redis.html)
