---
title: "關於 Redis Persistence - 資料持久化"
date: 2017-03-23T02:44:34+08:00
lastmod: 2018-09-14T00:42:34+08:00
draft: false
tags: ["Redis"]
slug: "redis-persistence"
aliases:
    - /2017/03/redis-persistence.html
---
# 關於 Redis Persistence - 資料持久化
我們知道 Redis 是個以 memory 為基礎的儲存系統，可以用來做為 database、cache 及 message broker。除了 cache 這個角色有較多情境可以忍受 server 有問題時造成資料遺失需要重新建立 cache 外，其他用途(database 及 message broker) 應該都無法接受資料遺失，因此 Redis 也發展出適合不同情境使用的持久化方式

## 持久化方式
1. RDB(Redis DataBasa file)
2. AOF(Append-Only File)

## 1. RDB(Redis DataBasa file)
- 優點
    - 在指定的時間間隔對資料做 snapshot 
    - 資料密度高，儲存特定時間點的資料內容 可以同時儲存不同時間區間的資料 
        
        > e.g. 每小時儲存過去 24 小時資料 or 每天儲存過去 30 天資料
    - 會 fork 出新 process 來進行資料儲存，fork 完成後 redis 主要 process 就不需其他 IO 操作
    - 相較於 AOF ，RDB 在 recovery 大量資料時速度較快
    - 可以指定特定時間內出現指定資料異動次數再建立 RDB
        
        > save {秒數} {資料異動次數}
- 缺點
    - 使用固定時間間隔進行儲存，可能會造成資料遺失
    - 資料量大時 fork 會耗用較多時間，可能會造成 Redis 有毫秒等級地無法服務，如果 server cpu 效能不好 可能會長達 1 秒無法服務
    
## 2. AOF(Append-Only File)
- 優點
    - fsync 策略，fsync 由 fork 出另個 process 進行
        - 停用 fsync
        - 每秒一次 (default)
        - 有寫入時 fsync
    - 只 append 資料，出現異常時可以隨時執行 redis-check-aof 來修復
    - 可以 rebuild AOF 來縮小 AOF 檔，過程中會持續對原 AOF 寫入資料，直到 新 AOF 成功建立後才會由新 AOF 進行服務
    - 預設 redis 重啟會優先載入 AOF
    - AOF 檔案內容容易閱讀與修改
    - 使用 `appendonly yes` 來啟用 AOF
- 缺點
    - 因為儲存所有寫入操作指令，相同數據量 AOF 檔案通常大於 RDB
    - 停用 fsync 可以讓 AOF 與 RDB 速度一樣快，其他策略 AOF 速度會慢於 RDB

## 列表比較
-|RDB|AOF
:---|---|---
工作原理|利用 copy-on-write 機制|利用 copy-on-write 機制
工作方式|1. fork 一份背景 process<BR/>2. 背景 PROCESS 將資料寫入臨時 RDB<BR/>3. 寫入完成後使用新 RDB 替換原 RDB 並刪除原有 RDB|1. fork 一份背景 process<BR/>2. 背景 process 將資料寫入臨時 AOF<BR/>3. 原 process 仍持續對原 AOF 寫入並將異動 cache<BR/>4. 背景 process 完成新 AOF 寫入後 通知主 process 將 cache 內異動加入新 AOF<BR/>5. 新舊 AOF 移轉
執行方式|指定的時間間隔對資料做 snapshot |根據 fsync 策略進行 <BR/> 1. 停用 fsync<br/>2. 每秒一次(default)<br/>3. 資料異動時
執行時間|特定時間點的資料 snapshot|一段時間內的資料修改歷程紀錄
檔案大小|較小|較大
備份速度|通常較快|通常較慢(停用 fsync 情況速度與 RDB 相同)
還原速度|較快|較慢
缺點|1. 有機會遺失備份間空檔的資料<BR/>2. 資料量大時 fork process 可能會造成 redis 短時間無法回應|1.檔案大小會不斷增加<br/>2.有多餘的操作指令紀錄
適用情境|特定時間點的整批備份<br/>用來進行災難還原|資料重要性高，確保資料安全性

# 參考資料
1. [Redis Persistence](https://redis.io/topics/persistence)





