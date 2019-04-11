---
title: "Redis 監控指標及監控工具"
date: 2017-03-21T02:43:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Tools","Redis"]
slug: "redis-monitoring"
aliases:
    - /2017/03/redis-monitoring.html
---
# Redis 監控指標及監控工具
 Redis (REmote DIctionary Server), 是一個 Key-Value 資料庫，跟 memcached 很類似，資料皆儲存在記憶體中，不同的是 Redis 對於資料儲存的限制比較少( key 再長 250 bytes，value 最大 1MB，時間 最長 30 天)，還可以設定定期地將資料儲存至硬碟。
 
 而為了達成 DevOps 與提高 SLA 系統及相關資源的監控都是必要的，今天就來看看該怎麼監控 redis 
 
##  Redis 監控警示的價值
1. redis 故障快速通知，定位故障點
2. 分析 redis 故障的 Root cause
3. redis 容量規劃和性能管理
4. redis 硬體資源利用率和成本


## 監控指標 1 (詳細資訊請參考 [How to Monitor Redis](https://blog.serverdensity.com/monitor-redis/))
1. 必要程序依預期執行
    
    衡量指標|說明|建議警示標準
    :---|:---|:---
    redis process|	常駐程式正確執行中|	redis 執行個數 != 1.
    uptime|	確保服務不會一直重複 |	uptime < 300s.

2. 系統資源使用不超出限制
    
    衡量指標|說明|建議警示標準
    :---|:---|:---
    Load|	all-in-one 效能指標. 高 load 會直接造成效能降低|load > cpu 核心數.
    CPU usage|	如果 cpu 使用率沒有滿百，可以忽略|	None
    Memory usage|記憶體使用量取決於儲存的資料量，但不該影響 os 的基本運作|None
    Swap usage|	Swap 是為了緊急情況使用的，其他情況不應使用，如果使用量有增加表示性能正在衰退中|used swap  > 128MB.
    Network bandwidth|流量與連線數及查詢量相關，通常只用來排除問題不需刻意警示|	None
    Disk usage|	確保有足夠的空間來儲存新資料、log、暫存檔案、snapshot 或是備份|	磁碟使用率 > 85%  

3. 成功執行查詢

    衡量指標|說明|建議警示標準
    :---|:---|:---
    connected_clients| 連線至 redis 的連線數|	connected_clients < 最小 application 數
    keyspace|key 總數量. 可用來與 hit_rate 比較來除錯.|	None
    instantaneous_ops_per_sec|	每秒處理的命令數|	None
    hit rate (calculated)|	keyspace_hits / (keyspace_hits + keyspace_misses)|	None
    rdb_last_save_time | 	上次儲存至磁碟的 timestamp(有啟用 persistence 才有用)|	rdb_last_save_time is > 3600 秒 (可接受的時間間隔)
    rdb_changes_since_last_save|上次儲存至磁碟後的修改量(暫存在 memory 重開後資料會不見)|	None
    connected_slaves|主要 redis 的 slave 數|	connected_slaves !=  cluster 中 slave 總數.
    master_last_io_seconds_ago|	 master 與 slaves 有多久沒互動|	master_last_io_seconds_ago is > 30 秒 (可接受的時間間隔)

4. 服務正確運作
     
    衡量指標|說明|建議警示標準
    :---|:---|:---
    latency	|redis 平均回應時間 	|latency > 200ms (最長可接受時間).
    used_memory	|redis 使用量<BR/>如果超過硬體限制就開始 swap 而造成效率大幅下降<BR/>可以使用 maxmeomry 來設定上限|	None
    mem_fragmentation_ratio|memory 碎片化比例，高碎片化比例會造成 swap 及效能低落|mem_fragmentation_ratio  > 1.5
    evicted_keys|因為 redis memory 使用量達到 maxmemory 時被刪除的 key 數量<BR/>如果被刪除的 ket 太多時表 latency 會增加|	None
    blocked_clients	|被 blocking 的連線數(BLPOP，BRPOP，BRPOPLPUSH 皆會造成 blocking)|None
    
5. 常見故障位置

    衡量指標|說明|建議警示標準
    :---|:---|:---
    rejected_connections|因為 maxclient 限制而被拒絕的連線數| rejected_connections > 你需要的連線數.
    keyspace_misses|	尋找 key 失敗的數量|只有在非 blocking 操作(BRPOPLPUSH, BRPOP and BLPOP), 且 keyspace_misses > 0.
    master_link_down_since_seconds|	master 與 slave 連接中斷的時間(以秒計)<BR/>如果重新連線會需要重新發送 sync 指令而影響效能|	master_link_down_since_seconds is > 60 秒.

## 監控指標 2 
詳細資訊請參考 [細說Redis監控和告警](https://zhuoroger.github.io/2016/08/20/redis-monitor-and-alarm/)，內容非常豐富，又是中文的就不另外 copy and paste 了


## 監控工具
1. [redis-cli info command](https://redis.io/commands/info)
    
    > redis 原生提供的指令，可以提供 redis server 的重要資訊及統計資訊
    
    - server:  redis server 相關資訊
    - clients: Client 端連線資訊
    - memory: Memory 使用狀況
    - persistence: 持久性 (RDB and AOF) 相關資訊
    - stats: 一般統計資訊
    - replication: Master/slave 複寫資訊
    - cpu: CPU 使用率資訊
    - commandstats: Redis commands 統計
    - cluster: Redis Cluster 資訊 (如果已啟動)
    - keyspace: Database (key 逾期) 相關統計資訊

2. [redis-cli monitor command](https://redis.io/commands/monitor)
    
    > redis 原生提供的指令，用來 debug ，會回傳所有 redis 收到的指令
    
    - 可以使用 `QUIT` 來停止 monitor 指令
    - 需要注意的是：因為回傳所有指令的關係，<span style="color:red">monitor 指令會影響 redis 執行效能</span>
    - 未使用 monitor
        
        ![1nomonitor](https://cloud.githubusercontent.com/assets/3851540/23451416/cf776a22-fe9a-11e6-8077-9acc96d75d18.png)
    - 使用 monitor
        
        ![2withmoniotr](https://cloud.githubusercontent.com/assets/3851540/23451417/cf807b62-fe9a-11e6-81e4-e5587534abb8.png) 
    - 依我自行測試的結果(如上兩張圖)，並沒有非常明顯的差異，但大陸網友測試在某些特定條件，可能會降低 50% 吞吐量，如果想要仔細瞭解可以參考 [這裡](http://redis.cn/commands/monitor.html)

3. [redis-stat](https://github.com/junegunn/redis-stat)
    - Ruby 撰寫
    - 使用 info 指令來進行監控 (相對 monitor 不影響效能)
    - windows 安裝及使用方式請參考 [如何在 Windows OS 上架設 Redis 監控工具 redis-stat](//blog.yowko.com/2017/03/redis-stat-on-windows.html)

4. [RedisLive](https://github.com/nkrode/RedisLive)
    - Python 撰寫
    - 有 google jsapi dependency，會自動連線 google 線上服務，有大陸網友自製拔除 jsapi 的版本，請參考 [GitHub](https://github.com/LittlePeng/redis-monitor)
    - 相關使用可以參考 [Redis監控工具,命令和調優](http://blog.csdn.net/dc_726/article/details/47699739)

5. [opserver](https://github.com/opserver/Opserver)
    - .NET 撰寫
    - 也是使用 info 指令
    - 安裝及使用方式請參考 [如何使用 Opserver 監控 Redis](//blog.yowko.com/2017/03/opserver-redis.html)

 
# 參考資料
1. [How to Monitor Redis](https://blog.serverdensity.com/monitor-redis/)
2. [Redis監控工具—Redis-stat、RedisLive](http://wxmimperio.tk/2016/02/25/Redis-Monitor-Tools/)
3. [細說Redis監控和告警](https://zhuoroger.github.io/2016/08/20/redis-monitor-and-alarm/)