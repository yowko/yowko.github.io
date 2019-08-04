---
title: "MongoDB CPU High 問題追查紀錄"
date: 2019-08-04T21:30:00+08:00
lastmod: 2019-08-04T21:30:31+08:00
draft: false
tags: ["MongoDB","Performance Tuning"]
slug: "mongodb-cpu-high"
---

## MongoDB CPU High 問題追查紀錄

在之前筆記 [取得 MongoDB SDK 實際產生的指令](https://blog.yowko.com/mongodb-interceptor/) 中提到是為了追查 MongoDB 的效能問題而需要取得 MongoDB driver 實際產生的 script，既然 script 已經順利取得，就可以來看看 MongoDB 到底在忙什麼了 ~~

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. MongoDB 4.0.10
4. NuGet Package

    - MongoDB.Driver 2.8.1

## 追查開始

1. 取得造成問題的 script

    > 某個層面來說，這是最難的一步，如果交易量龐大，command 也會很多，很難定位出造成問題的 script

    - 從 log 來找出可能的 script

        > c# driver 可以參考 [取得 MongoDB SDK 實際產生的指令](https://blog.yowko.com/mongodb-interceptor/) 來紀錄 script

    - 當下正在執行的指令

        > 連線至 MongoDB, 並執行 `db.currentOp()`，極有可能會取得錯誤資訊

    - 從 MongoDB slow query 中取得

        > 詳細內容請參考官方文件 [profile — MongoDB Manual](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)

2. 使用 `explain` 進行分析

    >`explain` 提供查詢資訊、使用索引以及查詢統計... 可以幫助釐清 command 在 MongoDB 中執行可能採用的執行計劃以利進一步的索引調校
    
    - 以下使用個人錄到的 command 為例

        ```json
        "command":{
            "q" : {
                "displaykey" : "key_9543_3"
            },
            "u" : {
                "$setOnInsert" : {
                "displaykey" : "key_9543_3",
                "created" : ISODate("2019-08-04T13:13:03.004Z")
                },
                "$set" : {
                "updated" : ISODate("2019-08-04T13:13:03.004Z"),
                "language" : [
                    {
                    "langcode" : "en-gb",
                    "displayvalue" : "key_9543_3"
                    }
                ]
                }
            },
            "multi" : false,
            "upsert" : true
            }
        ```

    - 使用 explain 分析

        > 連線至 MongoDB 執行，以下指令用來分析 `update` MongoDB 的操作，其中 explain 中的 `update` 為目標 collection 名稱，`updates` 則為欲分析的 script

        ```
        db.runCommand(
        {
            explain: {
                update: "localization",
                updates: [
                {
                "q" : {
                    "displaykey" : "key_9543_3"
                },
                "u" : {
                    "$setOnInsert" : {
                    "displaykey" : "key_9543_3",
                    "created" : ISODate("2019-08-04T13:13:03.004Z")
                    },
                    "$set" : {
                    "updated" : ISODate("2019-08-04T13:13:03.004Z"),
                    "language" : [
                        {
                        "langcode" : "en-gb",
                        "displayvalue" : "key_9543_3"
                        }
                    ]
                    }
                },
                "multi" : false,
                "upsert" : true
                }
                ]
            }
        }
        )
        ```

3. 分析結果

    ![1explain](https://user-images.githubusercontent.com/3851540/62425001-bffa8b00-b708-11e9-9262-d5f92fe33576.png)

    上圖中的 winningPlan 為指令分析後產生出最適合的執行計劃，操作 (stage) 為 `UPDATE`，而問題的徵結就出現在 inputStage (child stage) 中，可以看到 inputStage 採用 `COLLSCAN` (full collection scan) 

4. 解決方式

    > 加上適合的 index，重新進行 explain 分析就會發現改用 `IXSCAN` (index scan)

    ![2ixscan](https://user-images.githubusercontent.com/3851540/62425002-bffa8b00-b708-11e9-971f-1f68d0a74229.png)

## 心得

如果一開始效能監控沒開或是該加的 log 沒加，在 tuning 時就只能瞎子摸象，任何懷疑都只能用猜的，但該做哪些監控、加哪些 log 其實也都是經驗的累積，不同使用情境會有不同的做法

假設剛好在問題發現當下，直接執行 `db.currentOp()` 就可以得到足夠的資訊 - `command` 與 `planSummary`

![3plansummary](https://user-images.githubusercontent.com/3851540/62425003-bffa8b00-b708-11e9-9095-58960f1221fd.png)

運氣有沒有這麼好還是得看個人造化，所以我自己早就認清該紮實學習才能生存呀 XD

## 參考資訊

1. [取得 MongoDB SDK 實際產生的指令](https://blog.yowko.com/mongodb-interceptor/)
2. [profile — MongoDB Manual](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)
3. [explain](https://docs.mongodb.com/manual/reference/command/explain/)
4. [MongoDB干货系列2-MongoDB执行计划分析详解（1）](http://www.mongoing.com/eshu_explain1)
5. [MongoDB干货系列2-MongoDB执行计划分析详解（2）](http://www.mongoing.com/eshu_explain2)
6. [MongoDB干货系列2-MongoDB执行计划分析详解（3）](http://www.mongoing.com/eshu_explain3)
7. [6 Useful Tools to Monitor MongoDB Performance](https://www.tecmint.com/monitor-mongodb-performance/)
8. [MongoDB Performance](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#database-profiling)
