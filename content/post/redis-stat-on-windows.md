---
title: "如何在 Windows OS 上架設 Redis 監控工具 redis-stat"
date: 2017-03-04T01:42:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Redis"]
slug: "redis-stat-on-windows"
aliases:
    - /2017/03/redis-stat-on-windows.html
---
# 如何在 Windows OS 上架設 Redis 監控工具 redis-stat
Redis 官網有提到 `MONITOR` 命令會傳回所有的命令所以會影響效能，而大陸網友則提到某些特殊條件下最多可能會降低 50% 的吞吐量，影響甚距呀，因此想找找使用 `redis-cli info` 指令的監控工具，而 redis-stat 便是其一，加上 [redis-stat(GitHub)](https://github.com/junegunn/redis-stat) 有提到支援 Windows，試過後有些眉眉角角，趕緊紀錄一下。


## 安裝 JRuby
- `redis-stat` 是由 Ruby 撰寫，Windows 環境下透過 JRuby 來安裝是比較方便的

1. 下載對應版本(*.exe)
     
     ![1jruby](https://cloud.githubusercontent.com/assets/3851540/21705517/5b2da5f6-d3fb-11e6-896a-c464e505a722.png)
2. 執行安裝
    - 2-1. 點擊 exe 檔 進行安裝
        
        ![2-1install](https://cloud.githubusercontent.com/assets/3851540/21705514/5b26a2b0-d3fb-11e6-9f0c-69044f77e6ac.png)
    - 2-2. 選擇安裝目錄
        
        ![2-2folder](https://cloud.githubusercontent.com/assets/3851540/21705520/5b49b250-d3fb-11e6-8c68-d6488c43102b.png)
    - 2-3. 是否加入環境變數(建議)
        
        ![3path](https://cloud.githubusercontent.com/assets/3851540/21705512/5b224d0a-d3fb-11e6-81b8-3932efba7f0a.png) 

3. 確認是否成功安裝
    
    >`jruby -v`
    
    ![3installed](https://cloud.githubusercontent.com/assets/3851540/21705519/5b471658-d3fb-11e6-8b4f-db45e7e13327.png)    

## 安裝 redis-stat
> 參數清單可以參考 [JRubyCommandLineParameters](https://github.com/jruby/jruby/wiki/JRubyCommandLineParameters)

1. 開啟 command prompt
2. 切換至 JRuby 安裝目錄的 `bin` 資料夾
3. 使用 jruby 安裝
    - without proxy
        
        ```
        jruby -S gem install redis-stat
        ```
    - with proxy
        
        ```
        jruby -S gem install --http-proxy http://username:password@proxyserver:port redis-stat
        ```
        !![4redis-statinstalled](https://cloud.githubusercontent.com/assets/3851540/21705513/5b24e722-d3fb-11e6-819d-9d44f020595c.png)
4. 確認是否安裝成功
    
    ```
    redis-stat --version
    ```
    ![5installed](https://cloud.githubusercontent.com/assets/3851540/21705515/5b283346-d3fb-11e6-9440-9e627f9ea885.png)

## 啟動 redis-stat
1. 開啟 command prompt
2. 使用 `redis-stat`
    
    ```
    redis-stat [HOST[:PORT][/PASS] ...] [INTERVAL [COUNT]]
    ```
    
    參數|說明
    ---|:---
    -a, --auth=PASSWORD| redis 密碼
    -v, --verbose       |取得較多資訊
        --style=STYLE    |輸出格式 (unicode｜ascii)
        --no-color        |不使用 ANSI 顏色 (windows 下本來就沒有顏色)
        --csv=OUTPUT_CSV_FILE_PATH|將結果存成 CSV
        --es=ELASTICSEARCH_URL     | 將結果傳送到 ElasticSearch: [http://]HOST[:PORT][/INDEX]
        --server[=PORT] | 啟動 web 介面(預設使用 `63790` )
        --daemon         | Daemonize redis-stat. Must be used with --server option.<span style="color:red">這個選項試不出效果，看說明也不知道是做什麼的</span>
        --version   |  顯示 redis-stat 版本
        --help       | 顯示參數清單及說明
3. 開始監控
    - console
        
        ![6console](https://cloud.githubusercontent.com/assets/3851540/21705518/5b446638-d3fb-11e6-8f9c-03501c68374f.png) 
    - web 
        
        ![7web](https://cloud.githubusercontent.com/assets/3851540/21705516/5b29acbc-d3fb-11e6-9c63-fdb9dc3a410a.png)

## 心得
1. 使用 redis-cli info 效能較佳，不用擔心影響效能
2. 安裝上還算容易
3. 參數有些無法使用 (e.g. `--daemon`,`no-color`)
4. 無法 ctrl+c 停止

# 參考資料
1. [MONITOR(redis.io)](https://redis.io/commands/monitor)
2. [redis-stat(GitHub)](https://github.com/junegunn/redis-stat)
3. [JRuby.org](http://jruby.org/)