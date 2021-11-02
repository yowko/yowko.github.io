---
title: "建立 MongoDB 自訂角色權限 (role)"
date: 2020-09-08T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["MongoDB"]
slug: "mongodb-custom-role"
---

## 建立 MongoDB 自訂角色 (role)

最近在為系統的 storage 服務加上 data archival 流程，其中 MongoDB 的做法跟一般流程不同，雖然 MongoDB 有許多內建的角色權限 ([Built-In Roles](https://docs.mongodb.com/manual/reference/built-in-roles))，但沒找到適合的，我的需求是：

1. 需要可以跨 database 備份資料、建立 database、建立 collection、搜尋
2. 不能有破壞性操作 (dropCollection ....etc)

所以就興起使用 MongoDB [User-Defined Roles](https://docs.mongodb.com/manual/core/security-user-defined-roles/) 的念頭

因為我對於 MongoDB 的使用上一直以來都不是很得應手，所以趁著還有印象，簡單紀錄一下  避免日後又想不起

## 基本環境說明

1. macOS Catalina 10.15.6
2. docker desktop community 2.3.0.4(46911)
3. mongodb 4.2.8
4. MongoDB Replica Set 建立請參考之前筆記 [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)

## 設定方式

MongoDB 的 role 權限系統的設定邏輯是在指定 `resources` (Database , Collection or Cluster Resources) (參考資料：[Resource Document](https://docs.mongodb.com/manual/reference/resource-document/#resource-document)) 上加上想要需要的 `actions` (參考資料：[Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions)) 來達成的，除此之外也可以透過繼承其他 role 來給予權限

1. 先切換至 `admin` database

    ```bash
    use admin
    ```

2. 使用 `dbAdmin` 角色 user 進行認證

    - 語法

        ```bash
        db.auth('{username}','{password}')
        ```

    - 範例

        ```bash
        db.auth('root','pass.123')
        ```

3. 建立 User-Defined Role

    - 語法

        ```bash
        db.createRole(
           {
             role: "{role name}",
             privileges: [
               { resource: { db: "{target db name}", collection: "{target collection name}" }, actions: ["{action name}"] }
             ],
             roles: [
               { role: "{parent role name}", db: "{parent db name}" }
             ]
           },
           { w: "majority" , wtimeout: 5000 }
        )
        ```

    - 範例

        ```bash
        db.createRole(
           {
             role: "AchivalUser",
             privileges: [
               { resource: { db: "", collection: "" }, actions: [ "find","createIndex","createUser","insert" ] }
             ],
             roles: [
               { role: "backup", db: "admin" }
             ]
           },
           { w: "majority" , wtimeout: 5000 }
        )
        ```

    - 說明

        - 建立一個名為 `AchivalUser` 的 User-Defined Role
        - 對所有 database 與 collection 給于 `find`,`createIndex`,`createUser`,`insert` 權限
        - 繼承 `admin` 中的 `backup` role 權限
        - 需要同時得到 `1` primary 與 `1` secondary 的回應(replica set 下 `majority` 的定義)，回應 timeout 為 `5000`

    - 最終權限

        - find
        - createIndex
        - createUser
        - insert
        - create

            > create collection or view

        - listDatabases

            > from `backup` role

        - listCollections

            > from `backup` role

        - listIndexes

            > from `backup` role

4. 使用 User-Defined Role 建立 user

    - 語法

        ```bash
        db.createUser({user: "{username}" , pwd: "{password}", roles: [  "{role name}" ]})
        ```

    - 範例

        ```bash
        db.createUser({user: "archival" , pwd: "pass.123", roles: [  "AchivalUser" ]})
        ```

## 心得

每次處理 MongoDB 相關功能或是設定時，都有差不多的感覺：執行前很困難、資料不好找、文件語意不清；執行後異常簡單，不需要紀錄XD

這次也一樣，之前看相關需求時可謂是完全沒有頭緒，但真的測試搞懂後，連文件好像都覺得寫得很平易近人 @@"   我覺得應該是還沒有真正以 MongoDB 的思維模式來看的關係，但沒關係還是又過了一關

## 參考資訊

1. [Built-In Roles](https://docs.mongodb.com/manual/reference/built-in-roles)
2. [User-Defined Roles](https://docs.mongodb.com/manual/core/security-user-defined-roles/)
3. [使用 Docker Compose 建立有 Auth 的 MongoDB Replica Set](/docker-compose-mongodb-replica-set-with-auth/)
4. [Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions)
5. [Resource Document](https://docs.mongodb.com/manual/reference/resource-document/)
6. [Write Concern](https://docs.mongodb.com/manual/reference/write-concern/)
7. [Built-In Roles](https://docs.mongodb.com/manual/reference/built-in-roles)
