---
title: "Windows 平台上安裝 Elastic Stack (ELK - Elasticsearch , Logstash , Kibana)"
date: 2018-05-02T02:30:00+08:00
lastmod: 2018-10-06T13:46:31+08:00
draft: false
tags: ["ELK","Tools","Windows 10","Windows Server 2016"]
slug: "elastic-stack-elk-windows"
aliases:
    - /2018/05/elastic-stack-elk-windows.html
---
# Windows 平台上安裝 Elastic Stack (ELK - Elasticsearch , Logstash , Kibana)

**Elastic Stack (ELK - Elasticsearch , Logstash , Kibana) 建議執行在 Linux 平台上效能較佳**

`ELK Stack` 在 `5.0` 版本加入 `Beats` 套件後更名為 `Elastic Stack`，在還沒搞清楚 `Beats` 反而是 `5.5` 版本的變更引起我更大的關注：可以使用 `.msi` 在 Windows 環境上進行安裝，以往在 Windows 平台上安裝 ELK 都需要有一層轉換的動作，而透過 .MSI 安裝整體的體驗就比較接近 Windows 原生軟體，但 Java 8 的 runtime 還是必要的

剛好最近需要將手上的一些系統資料餵進 ELK ，剛好趁這個機會重新安裝 ELK 並且紀錄一下流程

## 安裝 Elasticsearch  

> 記得安裝 jre

1. 下載 [Elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch) 的 .msi 安裝檔
2. 執行 .msi 進行安裝
    - Locations
        
        > 設定 data , log 跟 config 的位置

        ![1locations](https://user-images.githubusercontent.com/3851540/39487060-9e221d6c-4db0-11e8-9a09-92ae4a5a31b3.png)
    - Service
        
        > 設定執行方式及執行身份

        ![2service](https://user-images.githubusercontent.com/3851540/39487062-9e6992e6-4db0-11e8-8f28-1f5132b75985.png)
    - Configuration
        
        > 設定 Elasticsearch 的識別資訊、使用 Memory 及網路

        ![3config](https://user-images.githubusercontent.com/3851540/39487063-9e9496bc-4db0-11e8-8849-f1fbe53074b0.png)
    - Plugins
        
        > 安裝其他套件

        ![4plugins](https://user-images.githubusercontent.com/3851540/39487064-9ebcbc8c-4db0-11e8-8a2a-24b0d61862c9.png)
    
    >![5installing](https://user-images.githubusercontent.com/3851540/39487065-9ee6635c-4db0-11e8-9b3b-a43813021ede.png)

    >![6success](https://user-images.githubusercontent.com/3851540/39487066-9f11878a-4db0-11e8-90e4-aeff3b0251e1.png)
3. 透過 http://localhost:9200/ 確認完成安裝
    
    ![7confirm9200](https://user-images.githubusercontent.com/3851540/39487067-9f3baa88-4db0-11e8-93af-d355c7e459ac.png)

## 安裝 Logstash
1. 下載並解壓縮 [Logstash](https://www.elastic.co/downloads/logstash)
2. 建立 `logstash.conf`
    
    ```yml
    input { stdin { } }
    output {
    elasticsearch { 
        hosts => ["localhost:9200"] 
        index => "yowkoindex"  
        }
    stdout { codec => rubydebug }
    }
    ``` 
3. 使用 `logstash.conf` 執行 logstash
    
    ```cmd
    "C:\ELK\logstash-6.2.4\bin\logstash.bat" -f "C:\ELK\logstash-6.2.4\config\logstash.conf"
    ``` 
4. 確認正確執行
    - console 輸出
        
        ![10logstashok](https://user-images.githubusercontent.com/3851540/39487071-9fbca156-4db0-11e8-9306-cd4d46555c86.png) 
    - 透過 http://localhost:9600/ 確認
        
        ![11logstashok](https://user-images.githubusercontent.com/3851540/39487072-9fe4d59a-4db0-11e8-854c-28dee8e40c46.png) 


## 安裝 Kibana 
1. 下載 [Kibana](https://www.elastic.co/cn/downloads/kibana) (只有 .zip 檔案)
2. 解壓縮並執行 `/bin/kibana.bat`
    
    > 看到 Status changed from yellow to green - Ready 就可以了

    ![8statuschanged](https://user-images.githubusercontent.com/3851540/39487068-9f671c9a-4db0-11e8-93a4-4ba7c9b5ac4e.png) 
3. 透過 http://localhost:5601 確認正常運作
    
    ![9kibanaok](https://user-images.githubusercontent.com/3851540/39487069-9f905c9a-4db0-11e8-97fd-a8f4eefac877.png)

## 測試結果
1. 在 logstash 上發送訊息
    - 在 logstash console 輸入 `yowko console` 
    - 收到 console 回應如下
        
        ```
        {
            "@version" => "1",
                "host" => "WIN-BACTHOVIL49",
            "message" => "yowko test elk\r",
            "@timestamp" => 2018-05-01T17:59:49.689Z
        }
        ``` 
    
    >![12logstashsend](https://user-images.githubusercontent.com/3851540/39487073-a00fb88c-4db0-11e8-8819-6b2155355ca7.png)
2. 開啟 kibana 的 dev tool 並在 console 進行查詢
    
    >![13kibanadevtool](https://user-images.githubusercontent.com/3851540/39487074-a039f1ce-4db0-11e8-887c-ea80b80c8e23.png)
3. 取得 index 結果
    
    >![14result](https://user-images.githubusercontent.com/3851540/39487075-a064f7e8-4db0-11e8-954c-ad70a4d62688.png) 

## 心得
Elasticsearch 的 Windows msi 安裝檔，使用起來比去過去的逐步安裝友善不少，只是 logstash 與 kibana 的安裝方式仍與過去差異不大，不過看得出 ELK 公司的投入，這幾年版本更新快速，功能也日益健全，過去我對於 Elasticsearch 一直停留在 search engine 的用途，近期發現很多方便的套件可以讓使用更為便利，除此之外速度也有所提升，難怪這幾年是非常熱門的工具

# 參考資訊
1. [Installing the Elastic Stack on Windows](https://logz.io/blog/elastic-stack-windows/)
2. [Installing the ELK Stack on Windows](https://logz.io/blog/installing-the-elk-stack-on-windows/)
3. [configure Logstash](https://www.elastic.co/guide/en/logstash/current/configuration.html#configuration)
4. [Elasticsearch 入門：logstash 5.0.0 安裝及輸出數據到 elasticsearch](https://blog.csdn.net/kk185800961/article/details/54430493)
