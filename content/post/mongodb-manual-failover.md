---
title: "手動觸發 MongoDB failover"
date: 2020-06-07T15:30:00+08:00
lastmod: 2020-12-11T15:30:31+08:00
draft: false
tags: ["MongoDB"]
slug: "mongodb-manual-failover"
---

## 手動觸發 MongoDB failover

最近如火如塗地準備著產品上線工作，其中一項重要項目就是各個服務的 HA 的驗證

我自己的測試方式是輪流將服務的每個 instance 停掉再重啟，確保服務中的每個 instance 都可以正常擔任主要提供服務的角色，也能正確地從主要角色卸離再重回輔助角色

停止服務可以分為：

1. server level (服務運作的實體機器無法提供服務運行：停機 or 網路斷線)

    > 驗證方式：機器 reboot 或是拔除網路線

2. service level (服務異常：資源不足、服務本身出現錯誤....)

    > 驗證方式：停止 service process 或是 使用 service 指令執行

今天紀錄一下怎麼透過 mongodb 指令來停止服務，以及如何強制 replica set 進行 primary 的切換 (這個動作比較像是將 mongodb replica set 調整回安裝時的角色分配，方便觀察 mongodb 是否有 failover 的狀況)

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker images

    - MongoDB 4.2.6

        > replica set 的建立請參考之前筆記 [Docker Compose 建立 MongoDB Replica Set](/docker-compose-mongodb-replica-set/)

## MongoDB 停止服務指令

> 這邊僅紀錄指令，如果搭配上面的 docker-compose 會出現 auth 的問題，這個日後有 機會再出一版帶有 auth 的 docker-compose

- 語法

    ```bash
    mongo [--host {mongo host}] [--port {mongo port}] [-u {username}] [-p {password}]  --eval "db.getSiblingDB('admin').shutdownServer()"
    ```

- 範例

    ```bash
    mongo --host 127.0.0.1 --port 6379 -u yowko -p pass.123  --eval "db.getSiblingDB('admin').shutdownServer()"
    ```

- unauth 的錯誤訊息

    ![1unauth](https://user-images.githubusercontent.com/3851540/83965358-c38d0e80-a8e5-11ea-854f-a8f199534640.jpg)

## 不停止服務執行 Failover

- 語法

    > 這需要對 primary 執行

    ```bash
    mongo [--host {mongo host}] [--port {mongo port}] [-u {username}] [-p {password}]  --eval "rs.stepDown()"
    ```

- 範例

    ```bash
    mongo --eval "rs.stepDown()"
    ```

- 不是 primary 改用 replica set 連線

    > 不是 primary 的錯誤訊息

    ![2notprimary](https://user-images.githubusercontent.com/3851540/83965359-c556d200-a8e5-11ea-83d5-c6ae5e13820b.jpg)

    > 詳細說明可以參考之前筆記 [MongoDB Cli Replica Set 連線方式](/mongodb-cli-replica-set/)

    ```bash
    mongo "mongodb://localhost:27017,localhost:27027,localhost:27037/?replicaSet=rs0" --eval "rs.stepDown()"
    ```

    
## 心得

雖然跟 MongoDB 交手也幾年了，但總覺得 MongoDB 還是很陌生，當然我自己沒有特別花時間去徹底了解它、熟悉它是我的問題，不過其他工具在使用上就不會遇到像 MongoDB 這樣老覺得陌生的狀況

像是 MongoDB auth 這點就是我一直以來的疑問，這麼多年來，設定方式始終如一 : 不是很便利，幸虧現在同事對 MongoDB 很熟悉，讓團隊在使用上沒有遇到太多問題

## 參考資訊

1. [Docker Compose 建立 MongoDB Replica Set](/docker-compose-mongodb-replica-set/)
2. [MongoDB Cli Replica Set 連線方式](/mongodb-cli-replica-set/)
3. [How to stop mongo DB in one command](https://stackoverflow.com/a/11777141)
4. [db.shutdownServer()](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/)
5. [rs.stepDown()](https://docs.mongodb.com/manual/reference/method/rs.stepDown/)
