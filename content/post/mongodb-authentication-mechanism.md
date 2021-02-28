---
title: "確認 MongoDB 驗證機制"
date: 2021-02-28T09:30:00+08:00
lastmod: 2021-02-28T09:30:31+08:00
draft: false
tags: ["MongoDB"]
slug: "mongodb-authentication-mechanism"
---

## 確認 MongoDB 驗證機制

前幾天在進行每日 server 狀態異常檢查時，偶然發現 MongoDB 的錯誤 log 很多，進一步整理後統整出每個 application 連線認證都會先出現一次 `SCRAM-SHA-256` 認證失敗之後才會出現 `SCRAM-SHA-1` 認證成功的 log 紀錄，今天就來紀錄一下當時追查的過程與語法

## 基本環境說明

1. macOS Big Sur 11.2.1
2. docker desktop 3.1.0(51484)
3. docker images

    - mongo:4.4.4-bionic

## 檢查語法

1. 指定連線驗證方式測試

    > 請參考官方文件 [Connection String URI Format](https://docs.mongodb.com/manual/reference/connection-string/#authentication-options)

    - 語法

        ```bash
        mongo  "mongodb[+srv]://[{username}:{password}@]{mongo host}:{mongo host}/?authMechanism=[SCRAM-SHA-1|SCRAM-SHA-256]"
        ```

    - 範例

        ```bash
        mongo  "mongodb://root:pass.123@127.0.0.1:27027/?authMechanism=SCRAM-SHA-256"
        ```

2. 檢查 mongo server 相容性設定

    > 請參考官方文件 [setFeatureCompatibilityVersion/default-values](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#default-values) 與 [setFeatureCompatibilityVersion/view-featurecompatibilityversion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-featurecompatibilityversion)

    ```js
    db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
    ```

    ![1viewfeaturecompatibilityversion](https://user-images.githubusercontent.com/3851540/109413999-7c9ef600-79eb-11eb-8c93-87f36aaf2d97.png)

3. 檢查 user 的 mechanisms

    - 3.1 單一 user

        - 語法

            ```js
            use admin;
            db.getUser("{username}")
            ```

        - 範例

            ```js
            use admin;
            db.getUser("test")
            ```

            ![2getuser](https://user-images.githubusercontent.com/3851540/109414001-7dd02300-79eb-11eb-8209-e578e6c50804.png)

    - 3.2 所有 user

        ```js
        use admin;
        db.getUsers();
        ```

        ![3getusers](https://user-images.githubusercontent.com/3851540/109414003-7e68b980-79eb-11eb-96b2-e7b663f388ed.png)

## 心得

原則上 MongoDB 4.0 之後，建立 user 時就會將 mechanisms 設定為 `["SCRAM-SHA-1","SCRAM-SHA-256"]`，但我們遇到狀況的環境是在 Mongo Atlas 上，經過與原廠確認：雖然 Mongo Atlas 的每個 instance 本身是 MongoDB 4.4 也支援 `SCRAM-SHA-256`，但 Mongo Atlas 服務並不支援 `SCRAM-SHA-256` XD，所以目前在 Mongo Atlas 只能使用 `SCRAM-SHA-1`

另外需要注意的是 Mongo Atlas 在不同 tier 上，可以執行的 admin command 不同，這個限制該讓問題追查難度更高

## 參考資訊

1. [Connection String URI Format](https://docs.mongodb.com/manual/reference/connection-string/#authentication-options)
2. [Configure MongoDB Authentication and Authorization](https://docs.cloudmanager.mongodb.com/tutorial/edit-host-authentication-credentials/#access-control-mechanisms)
3. [setFeatureCompatibilityVersion/default-values](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#default-values)
4. [setFeatureCompatibilityVersion/view-featurecompatibilityversion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-featurecompatibilityversion)
