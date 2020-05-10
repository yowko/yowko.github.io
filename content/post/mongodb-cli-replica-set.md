---
title: "MongoDB Cli Replica Set 連線方式"
date: 2020-05-09T21:30:00+08:00
lastmod: 2020-05-09T21:30:31+08:00
draft: false
tags: ["MongoDB"]
slug: "mongodb-cli-replica-set"
---

## MongoDB Cli Replica Set 連線方式

這是在建立 MongoDB patch 資料自動化流程時遇到的問題：需要進行 MongoDB data patch 的開發人員將需要執行的 MongoDB script 提供 `.js` 格式的檔案，再透過 MongoDB Cli 來匯入執行

從一開始的 single node 到後期的 MongoDB Replica Set 運作上都相當正常，最近因為網路問題有時會造成 MongoDB Replica Set failover 而出現錯誤;問題原因是沒有調整到 MongoDB Cli 匯入 script 所使用的連線資訊，簡單筆記一下，需要人工手動執行時才不用大費周章去查 jenkinsfile

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.5(43884)
3. MongoDB 4.2.6
4. show.js

    ```js
    show databases;
    ```

## 問題與解決方式

- 問題：未使用 replica set 來連線 failover 後無法正常執行

    - 連線方式：指定 `host` 與 `port`

        ```bash
        mongo -host localhost -port 27017 < show.js
        ```

    - 錯誤訊息

        ```txt
        2020-05-09T22:47:22.603+0800 E  QUERY    [js] uncaught exception: Error: listDatabases failed:{
        	"operationTime" : Timestamp(1589035642, 1),
        	"ok" : 0,
        	"errmsg" : "not master and slaveOk=false",
        	"code" : 13435,
        	"codeName" : "NotMasterNoSlaveOk",
        	"$clusterTime" : {
        		"clusterTime" : Timestamp(1589035642, 1),
        		"signature" : {
        			"hash" : BinData(0, "AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
        			"keyId" : NumberLong(0)
        		}
        	}
        } :
        ```

    - 錯誤截圖

        ![1error](https://user-images.githubusercontent.com/3851540/81478816-3d33cd00-9252-11ea-8d18-975e912231b7.jpg)

- 解決方式：使用完整連線字串

    - 連線方式

        ```bash
        mongo "mongodb://localhost:27017,localhost:27027,localhost:27037/?replicaSet=rs0" < show.js
        ```

    - 實際結果

        ![2result](https://user-images.githubusercontent.com/3851540/81478818-3efd9080-9252-11ea-8fcf-d8f6dbfe1809.jpg)

## 心得

一直以為 mongo cli 只能透過參數設定來連線，查了資料才發現原來可以使用完整連線字串，相當方便呀

## 參考資訊

1. [The mongo Shell](https://docs.mongodb.com/manual/mongo/)
