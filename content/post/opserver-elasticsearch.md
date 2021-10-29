---
title: "如何使用 Opserver 來監控 Elasticsearch"
date: 2017-03-28T01:00:00+08:00
lastmod: 2021-10-29T11:35:15+08:00
draft: false
tags: ["Monitoring","Tools","ELK"]
slug: "opserver-elasticsearch"
aliases:
    - /2017/03/opserver-elasticsearch.html
---
## 如何使用 Opserver 來監控 Elasticsearch

之前筆記 [如何使用 Opserver 來監控 Redis](/opserver-redis) 介紹該怎麼設定 Opserver for Redis，而 Opserver 官方介紹中也包含監控 Elasticsearch 的功能，立馬就來試試該如何設定

## 文章大綱

1. 安裝 Opserver
2. Opserver 安全性設定
3. 監控 Elasticsearch

## 安裝 Opserver

1. clone 專案
    * 從 [GitHub](https://github.com/opserver/Opserver) 下載(clone)

2. 解壓縮至硬碟
3. 設定啟始專案
    * 預設為 `Opserver.Core`

        ![1error](https://cloud.githubusercontent.com/assets/3851540/21705712/88ede1d0-d3fc-11e6-837c-d05d9d84f8cb.png)

    * `Opserver` 專案，按右鍵，選 `Set as StartUp Project`

        ![2startup](https://cloud.githubusercontent.com/assets/3851540/21705710/88ed648a-d3fc-11e6-9034-5ad2b8e8c919.png)

4. 編譯 (Build)

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

## 監控 Elasticsearch

* 加入 `ElasticSettings.json`
  * `Opserver` 專案, `Config` 資料夾下有 `ElasticSettings.json.example`

        ![1setting](https://cloud.githubusercontent.com/assets/3851540/21837616/3c977868-d808-11e6-895e-abfdcfb958ee.png)

  * rename `ElasticSettings.json.example` 為 `ElasticSettings.json`

* `ElasticSettings.json` 設定連線資訊
  * `clusters` 可以同時監控多 Elasticsearch instance
  * `name` 顯示名稱
  * `refreshIntervalSeconds` 自動重新取得資訊的間隔
  * `nodes` 可以有多台，可以填 servername 或是 ip，預設 port 是 `9200`

        ```json
        {
            "clusters": [
            {
                "name": "localhost",
                "refreshIntervalSeconds": 10,
                "nodes": [
                "serverName"
                ]
            }
            ]
        }
        ```

* 更新頻率(RefreshIntervalSeconds)預設值可以參考 `\Opserver.Core\Settings\ElasticSettings.cs`
  * 預設 `120 秒`

* Elasticsearch port 預設值 `\Opserver.Core\Data\Elastic\ElasticCluster.KnownNodes.cs`
  * 預設 port `9200`

## 監控結果

1. all cluster

    ![2all](https://cloud.githubusercontent.com/assets/3851540/21837615/3c9723c2-d808-11e6-82b3-feaadca8d2ca.png)

2. cluster

    ![3cluster](https://cloud.githubusercontent.com/assets/3851540/21837617/3c977f8e-d808-11e6-9805-11a337ccc0e7.png)

3. node
    ![4node](https://cloud.githubusercontent.com/assets/3851540/21837618/3c97f5d6-d808-11e6-91b4-c6ceeef24248.png)

4. indexes
    ![5indexes](https://cloud.githubusercontent.com/assets/3851540/21837619/3ca479c8-d808-11e6-81fa-cca59c3d8c40.png)

## Elasticsearch Indice

顏色|    說明
:---|:---
yellow|    代表沒有備份到 cluster
green|    代表 cluster 裡面所有 node 都有
red|    代表有問題，需要recovery

## 參考資訊

1. [GitHub](https://github.com/opserver/Opserver)
2. [如何使用 Opserver 來監控 Redis](/opserver-redis)
