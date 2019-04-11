---
title: "如何使用 Opserver 來監控 Windows Server"
date: 2017-03-29T01:00:00+08:00
lastmod: 2018-09-16T15:38:59+08:00
draft: false
tags: ["Monitoring","Tools"]
slug: "opserver-windows-server"
aliases:
    - /2017/03/opserver-windows-server.html
---
# 如何使用 Opserver 來監控 Windows Server
之前文章 [如何使用 Opserver 來監控 Redis](//blog.yowko.com/2017/03/opserver-redis.html) 介紹該怎麼設定 Opserver for Redis，[如何使用 Opserver 來監控 Elasticsearch](//blog.yowko.com/2017/03/opserver-elasticsearch.html) 介紹如何設定 Opserver for Elasticsearch ,接著就來看看該如何使用 Opserver 監控 Windows Server

## 文章大綱
1.  安裝 Opserver
2.  Opserver 安全性設定
3.  監控 Windows Server


## 安裝 Opserver
1. clone
    * 從 [GitHub](https://github.com/opserver/Opserver) 下載(clone)

2.  解壓縮至硬碟
3.  設定啟始專案
    * 預設為 `Opserver.Core`
        
        ![1error](https://cloud.githubusercontent.com/assets/3851540/21705712/88ede1d0-d3fc-11e6-837c-d05d9d84f8cb.png)

    * `Opserver` 專案，按右鍵，選 `Set as StartUp Project`
        
        ![2startup](https://cloud.githubusercontent.com/assets/3851540/21705710/88ed648a-d3fc-11e6-9034-5ad2b8e8c919.png)
4.  編譯 (Build)


## Opserver 安全性設定
* 沒找到 `SecuritySettings.config` 的錯誤
    ![3needconfig](https://cloud.githubusercontent.com/assets/3851540/21705708/88ab5a7c-d3fc-11e6-8b10-761fff9a2b89.png)

* 加入 `SecuritySettings.config`
    * `Opserver` 專案, `Config` 資料夾下有 `SecuritySettings.config.example`

        ![4securityconfig](https://cloud.githubusercontent.com/assets/3851540/21705714/88f073f0-d3fc-11e6-8d50-5c2448184ad6.png)

    * rename `SecuritySettings.config.example` 為 `SecuritySettings.config`
    * 依需求設定權限(`/about`可以檢查現行安全設定)
        * AD (default)
            
            >`<SecuritySettings provider="AD" />`

            ![5AD](https://cloud.githubusercontent.com/assets/3851540/21705716/89100170-d3fc-11e6-9dc7-8b3576ba93b4.png)

        * alladmin
            
            >`<SecuritySettings provider="alladmin" />`

            ![6alladmin](https://cloud.githubusercontent.com/assets/3851540/21705715/88f40ab0-d3fc-11e6-96bf-c8d44582fac3.png)

        * View All
            
            >`<SecuritySettings provider="" />`
            
            ![7viewall](https://cloud.githubusercontent.com/assets/3851540/21705717/89103334-d3fc-11e6-8a9b-b024b7c8e224.png)

    * 如果不是使用 `AD` ，畫面需要帳號密碼，請使用 `admin`/`admin`
        
        ![login](https://cloud.githubusercontent.com/assets/3851540/26041694/7a9ec8c2-3961-11e7-881c-b62d45245d6b.png)

## 監控 Windows Server

* 加入 `DashboardSettings.json`
    * `Opserver` 專案, `Config` 資料夾下有 `DashboardSettings.json.example`
        
        ![1setting](https://cloud.githubusercontent.com/assets/3851540/21882123/05ab390c-d8e4-11e6-8f0a-55fff50be407.png)

    * rename `DashboardSettings.json.example` 為 `DashboardSettings.json`

* `DashboardSettings.json` 設定連線資訊 `providers`
    * 以 `wmi` 為例
        *   `nodes` 可以有多台，可以填 servername 或是 ip
        *   `StaticDataTimeoutSeconds` 是靜態資料(e.g. node 名稱，及磁碟 size) cache 秒數，預設是 300 秒
        *   `DynamicDataTimeoutSeconds` 是動態資料 (e.g. CPU load) cache 秒數，預設是 30 秒
        *   `HistoryHours` 紀錄保留時間，預設 2 小時
        *   `Username` 遠端電腦的使用者帳號
        *   `Password` 達端電腦的使用者密碼
        
        ```json
        {
            "providers": {
                "wmi": {
                "nodes": [ "192.168.1.1","192.168.2.1" ],
                "staticDataTimeoutSeconds": 300,
                "dynamicDataTimeoutSeconds": 5,
                "historyHours": 2,
                "Username": "AD\\username",
                "Password": "password"
                }
            }
        }
        ```

* 靜態資料 cache 時間 (StaticDataTimeoutSeconds)
    
    > 預設值可以參考 `\Opserver.Core\Settings\DashboardSettings.WMI.cs`

    * 預設 `300 秒`

* 動態資料 cache 時間 (DynamicDataTimeoutSeconds)

    > 預設值可以參考 `\Opserver.Core\Settings\DashboardSettings.WMI.cs`

    * 預設 `30 秒`

* 紀錄保留時間 cahce 時間 (HistoryHours)

    > 預設可以參考 `\Opserver.Core\Settings\DashboardSettings.WMI.cs`

    * 預設 `2 小時`

- 設定警戒值

    key|	說明
    :---|:---
    cpuWarningPercent|	cpu 用量警戒(黃色)
    cpuCriticalPercent|	cpu 用量危急(紅色)
    memoryWarningPercent|	記憶體 用量警戒(黃色)
    memoryCriticalPercent|	記憶體 用量危急(紅色)
    diskWarningPercent|	磁碟空間 用量警戒(黃色)
    diskCriticalPercent|	磁碟空間 用量危急(紅色)

* 設定監控 server 分群
    *   利用 regular expression 解析 server name 來分群
    *   也可以依群組個別設定 警戒值
        
        ```json
        "categories": [
            {
            "name": "DEV Web Servers",
            "pattern": "192.168.1.*",
            "cpuWarningPercent": 25,
            "memoryWarningPercent": 65,
            "memoryCriticalPercent": 75
            },
            {
            "name": "Localhost Web Servers",
            "pattern": "192.168.2.*",
            "cpuWarningPercent": 25,
            "memoryWarningPercent": 75
            }
        ],
        ```

## 監控結果

![2result](https://cloud.githubusercontent.com/assets/3851540/21882124/05b131b8-d8e4-11e6-883d-a0f586767fde.png)

## 心得

只能使用同一組帳號密碼，比較難符合現實需求，但 open source 的好處就是不足的地方可以自行修改

## 20170612 補充： Server 離線

* Server 離線會以下特徵

    1.  主機名稱跟主機圖示會變成黃燈
    2.  該離線主機所有資訊也會加上黃底


    ![3offline](https://user-images.githubusercontent.com/3851540/27023331-ae67703a-4f84-11e7-9dd8-952907419c77.png)

# 參考資料

1.  [Opserver GitHub](https://github.com/opserver/Opserver)
2.  [如何使用 Opserver 來監控 Redis](//blog.yowko.com/2017/03/opserver-redis.html)
3.  [如何使用 Opserver 來監控 Elasticsearch](//blog.yowko.com/2017/03/opserver-elasticsearch.html)
