---
title: "MongoDB 在 Windows 上的 HA 機制 - Replica Sets"
date: 2017-09-05T01:19:00+08:00
lastmod: 2021-11-02T17:21:34+08:00
draft: false
tags: ["MongoDB","Windows Service"]
slug: "mongodb-windows-ha-replica-sets"
aliases:
    - /2017/09/mongodb-windows-ha-replica-sets.html
---
## MongoDB 在 Windows 上的 HA 機制 - Replica Sets

嚴格來說 MongoDB 在 HA 上支援過兩種作法：1. Master-Slave 2. Replica Set，但官網已正式公告從 MongoDB 3.2 起開始不再將 master-slave 視為 cluster 機制的一部份，詳細內容請參考官方文件 [Master Slave Replication](https://docs.mongodb.com/manual/core/master-slave/)，取而代之的是 Replica sets，因此現在建立 MongoDB HA 當然得採取 Replica sets

剛好趁公司要建立新 MongoDB HA 機制，重新筆記一下流程及相關操作

## 關於 Replica Set

Replica Set 是數個 MongoDB 的 instance，用來同時儲存相同資料，其中各個 MongoDB 依其作用有不同角色

1. Primary

    * 一個 Replica Set 中僅會有一個 Primary
    * 負責處理所有 `寫入` 操作，並將操作紀錄至 `oplog` 上

    ![Primary](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)

2. Secondary
    * 一個 Replica Set 中可以有一個以上 Secondary
    * 將 Primary 的 oplog 複製至自己的資料庫中

    ![Secondary](https://docs.mongodb.com/manual/_images/replica-set-primary-with-two-secondaries.bakedsvg.svg)

3. Arbiter

    * 一個 Replica Set 中可以有零個或一個 Arbiter
    * 不實際儲存資料內容，僅參與其他 Secondary 的心跳檢測與新 Primary 選舉，為的是讓選舉可以達成有效人數
    * 相對於 Secondary 因未參與資料複製而較節省硬體資源

    ![Arbiter](https://docs.mongodb.com/manual/_images/replica-set-primary-with-secondary-and-arbiter.bakedsvg.svg)

* Automatic Failover
  * 當 Primary 未回應其他成員的心跳檢測超過 10 秒，就會引發 failover
  * 會由多個 Secondary 中推選出一個新的 Primary

    ![Automatic Failover](https://docs.mongodb.com/manual/_images/replica-set-trigger-election.bakedsvg.svg)

## 建立 Replica Set

以下將會透過 `yaml` 格式建立 Windows Service 版的 MongoDB instance，如果不清楚做法請參考 [如何在 Windows 環境安裝及設定 MongoDB](/2017/08/windows-mongodb.html)

1. 建立 MongoDB 的 yaml 設定檔

    > 以下將會建立三個角色各一的 `yaml`，設定部份大同相異，需要注意的是加上了 `replication` 的部份，用來讓各個角色認識彼此

    * 1-1. Primary

        ```yml
        systemLog:
            destination: file
            path: "C:\\mongodb\\replicaset\\primary\\log.log"
            logAppend: true
        storage:
            dbPath: "C:\\mongodb\\replicaset\\primary"
            directoryPerDB: true
        net:
            port: 27017
            bindIp: 127.0.0.1
        processManagement:
            windowsService:
                serviceName: MongoDBPrimary
                displayName: MongoDBPrimary
        replication:
            replSetName: TestReplicaSet
        ```

    * 1-2. Secondary

        ```yml
        systemLog:
            destination: file
            path: "C:\\mongodb\\replicaset\\secondary\\log.log"
            logAppend: true
        storage:
            dbPath: "C:\\mongodb\\replicaset\\secondary"
            directoryPerDB: true
        net:
            port: 27027
            bindIp: 127.0.0.1
        processManagement:
            windowsService:
                serviceName: MongoDBSecondary
                displayName: MongoDBSecondary
        replication:
            replSetName: TestReplicaSet
        ```

    * 1-3. Arbiter

        ```yml
        systemLog:
            destination: file
            path: "C:\\mongodb\\replicaset\\arbiter\\log.log"
            logAppend: true
        storage:
            dbPath: "C:\\mongodb\\replicaset\\arbiter"
            directoryPerDB: true
        net:
            port: 27037
            bindIp: 127.0.0.1
        processManagement:
            windowsService:
                serviceName: MongoDBArbiter
                displayName: MongoDBArbiter
        replication:
            replSetName: TestReplicaSet
        ```

2. 安裝並啟動 MongoDB

    * 2-1. 安裝
        * Primary

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\mongodb\replicaset\primary\mongo.conf" --install
            ```

        * Secondary

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\mongodb\replicaset\secondary\mongo.conf" --install
            ```

        * Arbiter

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\mongodb\replicaset\arbiter\mongo.conf" --install
            ```

    * 2-2. 啟動

        * Primary

            ```cmd
            net start MongoDBPrimary
            ```

        * Secondary

            ```cmd
            net start MongoDBSecondary
            ```

        * Arbiter

            ```cmd
            net start MongoDBArbiter
            ```

3. 連線至 MongoDB Primary

    > 剛剛建立的三個 instance ，本身並沒有角色差異，只是為了辨識方便，硬給個名字而已，原則上隨便挑一個就可以，為避免混淆建議命名要有規則

    ```cmd
    "C:\Program Files\MongoDB\Server\3.4\bin\mongo.exe" 127.0.0.1:27017
    ```

    ![1connecttoprimary](https://user-images.githubusercontent.com/3851540/30034801-720a0898-91d6-11e7-88cd-40128a00c230.png)

4. 初始化 Replica Set

    > 指定 `replSetName` 並加入 primary

    ```js
    rs.initiate( {   _id : "TestReplicaSet",members: [ { _id : 0, host : "127.0.0.1:27017" } ]})
    ```

    ![2init](https://user-images.githubusercontent.com/3851540/30034802-7230f9f8-91d6-11e7-84cf-b84f8e004dd5.png)

    * 成功會回傳 `"ok":1`
    * 執行 enter 換行，會發現角色已變為 `Primary`

5. 加入 `Secondary`

    ```js
    rs.add("127.0.0.1:27027")
    ```

    ![3secondary](https://user-images.githubusercontent.com/3851540/30034803-724dbb2e-91d6-11e7-9406-01f5bdad3999.png)

6. 確認 Replica Set 狀態

    ```js
    rs.status()
    ```

    ![4rsstatus](https://user-images.githubusercontent.com/3851540/30034805-7254edf4-91d6-11e7-9df4-787adc58bcd8.png)

7. 加入 Arbiter

    ```js
    rs.addArb("127.0.0.1:27037")
    ```

    ![5addarbiter](https://user-images.githubusercontent.com/3851540/30034804-72544d86-91d6-11e7-96af-c54dba81b168.png)

8. 確認 Replica Set 狀態

    ```js
    rs.status()
    ```

    ![6arbiterstatus](https://user-images.githubusercontent.com/3851540/30034806-725832b6-91d6-11e7-8a13-9b515aa9f510.png)

9. 模擬 primary down 執行 failover

    * 在 primary 中執行下列指令

        * 切換至 `admin`

            ```js
            use admin
            ```

            ![7useadmin](https://user-images.githubusercontent.com/3851540/30034807-725e6ff0-91d6-11e7-81f9-b5760710d150.png)

        * 關閉 mongodb instance

            ```js
            db.shutdownServer()
            ```

            ![8shutdown](https://user-images.githubusercontent.com/3851540/30034808-725e858a-91d6-11e7-87dd-549990cfd6e3.png)

    * secondary 提為 primary

        ![9becomeprimary](https://user-images.githubusercontent.com/3851540/30034809-72753d52-91d6-11e7-96dd-de70bb5477b8.png)

## 心得

不知道什麼緣故，每次做完 MongoDB 的設定，我都有種 "滿容易的呀，這不用特別做筆記了吧！！"，但每次要設定時又會卡關一陣子XD

經過再次設定以及簡單紀錄後，我相信下次應該不會 ~~再忘記了吧~~，或是該說我知道哪邊可以查了

使用 Replica Set 設定 MongoDB HA，比 master-slave 方便，至少不用自行處理 master/slave 切換，設定上難度也不高，值得推薦

## 參考資訊

1. [Master Slave Replication](https://docs.mongodb.com/manual/core/master-slave/)
2. [關於MongoDb Replica Set的故障轉移集群——實戰篇](http://www.cnblogs.com/yaoxing/p/mongodb-replica-set-setup.html)
3. [Stop mongos process](https://serverfault.com/questions/746121/stop-mongos-process)
