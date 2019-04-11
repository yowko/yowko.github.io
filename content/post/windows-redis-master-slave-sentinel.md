---
title: "Windows 環境如何設定 Redis Master-Slave 與 Sentinel"
date: 2017-03-24T02:44:34+08:00
lastmod: 2018-09-14T00:42:34+08:00
draft: false
tags: ["Redis"]
slug: "windows-redis-master-slave-sentinel"
aliases:
    - /2017/03/windows-redis-master-slave-sentinel.html
---
# Windows 環境如何設定 Redis Master-Slave 與 Sentinel
最近在幫公司同事做 Redis 基礎教育，雖說公司 Redis 架在 Rehat 上，但身為一個 .NET 開發人員讓我轉換環境操作 Linux 還是相對苦手的，所以就在自己的 Windows 電腦上架了 Redis Master-Slave 與 Sentinel，隨手筆記備忘一下

## 準備工作
1. 下載 Windows 版 [Redis](https://github.com/MSOpenTech/redis/releases)
    - .msi 可以直接安裝為 Windows Service (<span style="color:red">但設定調整比較麻煩，建議使用 .zip</span>)
        
        > 這邊有安裝流程說明 [使用 Redis 當做 ASP.NET MVC 的 Session State Server](https://blog.yowko.com/2017/01/redis-aspnet-mvc-session-state-server.html)
        - 預設安裝在 `c:\Program Files\Redis` 檔案修改權限設定比較繁瑣
    - .zip 
        - 建議不要放在 OS 槽，會少掉許多權限設定問題
2. 準備建立 Master-Slave 與 Sentinel(下列方式擇一即可)
    - 採取實體隔離
        - 將 zip 解壓縮至資料夾後複製多份並標記用途(e.g. Redis-{Master|SlaveSentinel})後備用 
    - 僅建立不同用途之 config
3. 以下 demo 將以建立不同 config 方式進行

## Master-Slave 設定
* 調整 `redis.windows.conf` config 設定
    1. Master
        - 保留預設值即可
    2. Slave
        - 如果採用 config 隔離，請記得修改檔案(e.g. redis.windows6380.conf)
        - 修改 port (不可與 master port 重複) 
        - 加入 slave 設定
            
            > `slaveof {masterip} {masterport>}`
* 啟動 Master & Slave (下列方式擇一即可)
    1. 指令
        - 啟動 master 
            - 執行 redis-server.exe 並指定使用 master config
                
                >`redis-server redis.windows.conf`
                
                ![1master](https://cloud.githubusercontent.com/assets/3851540/24081661/f7317920-0cf2-11e7-9a10-522981ec0ec1.png)
        - 啟動 slave
            - 執行 redis-server 並指定使用 slave config (執行前確認 config 是否已正確修改)
                
                >`redis-server redis.windows6380.conf`
                
                ![2slave](https://cloud.githubusercontent.com/assets/3851540/24081662/f731e8ce-0cf2-11e7-8ad2-df1bde02c9ac.png)
            - 啟動後 Master 也會出現 sync 資料的訊息
                
                >![3slavesync](https://cloud.githubusercontent.com/assets/3851540/24081663/f73367f8-0cf2-11e7-985e-ed4e5bface89.png) 
    2. Windows Service
        * **如果路徑包含空白請記得使用跳脫字元 `\` 搭配 `"` 將路徑包起來，下面 Sentinel 有使用範例**
        - 設定 master
            
            ```
            sc.exe create "Redis6379" start= auto binPath= "c:\redis\redis-server.exe --service-run c:\redis\redis.windows-service.conf" DisplayName= "Redis6379"
            ```
        - 設定 slave
            
            ```
            sc.exe create "Redis6380" start= auto binPath= "c:\redis\redis-server.exe --service-run c:\redis\redis.windows-service6380.conf" DisplayName= "Redis6380"
            ```
* 確認執行狀況
    1. Master
        - `redis-cli -h 127.0.0.1 -p 6379 info replication`
            
            ![4masterinfo](https://cloud.githubusercontent.com/assets/3851540/24081664/f753ebd6-0cf2-11e7-980f-7ad6863fcfbd.png)
    2. Slave
        - `redis-cli -h 127.0.0.1 -p 6380 info replication` 
            
            ![5slaveinfo](https://cloud.githubusercontent.com/assets/3851540/24081665/f756656e-0cf2-11e7-88a0-8a681f9a9ff1.png)

## Sentinel config 設定
<span style='color:red'>Winodws 7 使用 Windows Redis 3.2.100 版本無法啟動 sentinel，需使用 3.0.504 版</span>

- 新增 sentinel.conf 檔案
- 逐一加入以下設定
    - 指定 port
        
        > port 26379
    - 指定監視名為 `master` ip 為 `127.0.0.1` port 為 `6379` 的 redis master instance，最後數字代表幾個 sentinel 同意才進行切換
        
        >sentinel monitor master 127.0.0.1 6379 1
    - 指定 `3000` 毫秒沒有回應就視名為 `master` 的 instance 失效
        
        >sentinel down-after-milliseconds master 3000
    - 指定 `18000` 毫秒未完成視為 failover 失敗
        
        >sentinel failover-timeout master 18000
    - 指定同時只有 `1` 個 salve 可以從新 master 同步資料回去
        
        >sentinel parallel-syncs master 1
        
        - 數字愈小，failover 耗時就愈久(需等所有 slave 都同步完)
        - slave 同步資料可能會造成 slave 無法回應，所以也不建議設太大
    - 指定連線密碼為 `password`(非必要)
        
        >sentinel auth-pass password
- 最終設定範例
    
    ```
    port 26379
    sentinel monitor master 127.0.0.1 6379 1
    sentinel down-after-milliseconds master 3000
    sentinel failover-timeout master 18000
    sentinel parallel-syncs master 1
    ```

## 啟動 Sentinel
1. 指令
    - 執行 redis-server.exe 並指定使用 sentinel config 及 `--sentinel` 參數 (執行前請確認 sentinel config 已設定)
        - `redis-server.exe sentinel.conf --sentinel `
            
            ![6sentinel](https://cloud.githubusercontent.com/assets/3851540/24081666/f7572cb0-0cf2-11e7-89d5-605cb3388828.png)
2. Windows Service
    * **如果路徑包含空白請記得使用跳脫字元 `\` 搭配 `"` 將路徑包起來**
        
        ```
        sc.exe create "RedisSentinel" start= auto binPath= "\"c:\Program Files\redis\redis-server.exe\" --service-run \"c:\Program Files\redis\sentinel.conf\" --sentinel" DisplayName= "RedisSentinel"
        ```

## 確認執行狀態
```
redis-cli -h 127.0.0.1 -p 26379 info sentinel
```
        
![7sentinelinfo](https://cloud.githubusercontent.com/assets/3851540/24081668/f758b76a-0cf2-11e7-8398-63ddd11b03cf.png) 

## 模擬 master 失效
1. 手動 shutdown master
    - `redis-cli -h 127.0.0.1 -p 6379 shutdown` 
        
        ![8mastershutdown](https://cloud.githubusercontent.com/assets/3851540/24081669/f75c7d78-0cf2-11e7-8c55-4c509d0cea63.png)
2. 確認 sentinel 資訊
    -  consoel 出現 failover 操作資訊
        
        ![9sentinelconsole](https://cloud.githubusercontent.com/assets/3851540/24081667/f758e79e-0cf2-11e7-8f24-a97fe1fe39a6.png)  
    -  config 也自動修改了
        
        ![10sentinelconfig](https://cloud.githubusercontent.com/assets/3851540/24081670/f778bcf4-0cf2-11e7-8420-9a2c1e19296a.png)
      
3. 確認 slave 資訊
    -  console 出現啟用 Master mode
        
        ![11slave2master](https://cloud.githubusercontent.com/assets/3851540/24081671/f77c49d2-0cf2-11e7-8166-87e986ab57c0.png)  
    -  config
        
        > slaveof {masterip} {masterport>} 已被移除
    - 使用 `redis-cli -h 127.0.0.1 -p 6380 info replication` 確認
        
        ![12slave2master](https://cloud.githubusercontent.com/assets/3851540/24081672/f77c805a-0cf2-11e7-9dc3-ffc611ac4490.png)

## 模擬 slave 接替成為 master 後，原 master 回復服務

1. 手動 shutdown master
    - `redis-cli -h 127.0.0.1 -p 6379 shutdown` 
        
        ![8mastershutdown](https://cloud.githubusercontent.com/assets/3851540/24081669/f75c7d78-0cf2-11e7-8c55-4c509d0cea63.png)
2. 確認 sentinel 資訊
    - consoel 出現 failover 操作資訊
        
        ![9sentinelconsole](https://cloud.githubusercontent.com/assets/3851540/24081667/f758e79e-0cf2-11e7-8f24-a97fe1fe39a6.png)  
    - config 也自動修改了
        
        ![10sentinelconfig](https://cloud.githubusercontent.com/assets/3851540/24081670/f778bcf4-0cf2-11e7-8420-9a2c1e19296a.png)
    
3. 確認 slave 資訊
    - console 出現啟用 Master mode
            
        ![11slave2master](https://cloud.githubusercontent.com/assets/3851540/24081671/f77c49d2-0cf2-11e7-8166-87e986ab57c0.png)  
    - config
            
        > slaveof {masterip} {masterport>} 已被移除
    - 使用 `redis-cli -h 127.0.0.1 -p 6380 info replication` 確認
            
        ![12slave2master](https://cloud.githubusercontent.com/assets/3851540/24081672/f77c805a-0cf2-11e7-9dc3-ffc611ac4490.png)

1. 回復已 shutdown 的 master
    - 再次啟動原 master
        - 自動連線至新 master (原 slave) ，並成為其 slave 開始同步資料
            
            ![13master2slave](https://cloud.githubusercontent.com/assets/3851540/24081656/f6ea3c22-0cf2-11e7-8545-97f923b1cb5b.png) 
    - 新 master (原 slave) 同步資料至 原 master(新 slave)
        
        ![14slavesynctomaster](https://cloud.githubusercontent.com/assets/3851540/24081657/f70f26fe-0cf2-11e7-9a30-1458c481adbf.png) 
    - Sentinel 偵測到`原 master`加入並將其指定為 slave
        
        ![15sentinelslave](https://cloud.githubusercontent.com/assets/3851540/24081658/f72f93b2-0cf2-11e7-9be5-88e18680fa34.png)
    - 原 master(新 slave) 的 config 會被自動加上 `slaveof {新 master} {新 masterport>}`
    - 使用指令確認一下
        - 原 master：6379 --> slave
            - `redis-cli -h 127.0.0.1 -p 6379 info replication`
                
                ![176379](https://cloud.githubusercontent.com/assets/3851540/24081659/f7303272-0cf2-11e7-912c-15499d40dfdb.png) 
        - 原 slave ：6380 --> master
            - `redis-cli -h 127.0.0.1 -p 6380 info replication`
                
                ![166380](https://cloud.githubusercontent.com/assets/3851540/24081660/f730d434-0cf2-11e7-8fbc-2f185aa43688.png)

# 參考資料
1. [Redis](https://github.com/MSOpenTech/redis/releases)
2. [Windows下Redis Sentinel部署（包含Redis Replication）](http://bbs.redis.cn/forum.php?mod=viewthread&tid=715)
2. [[料理佳餚] 使用Redis-Sentinel 打造Redis 的HA](https://dotblogs.com.tw/supershowwei/2016/02/03/123740)