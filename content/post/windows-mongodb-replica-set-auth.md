---
title: "在 Windows 啟用 MongoDB Replica Set 驗證"
date: 2017-09-10T23:13:00+08:00
lastmod: 2020-12-11T12:03:57+08:00
draft: false
tags: ["MongoDB"]
slug: "windows-mongodb-replica-set-auth"
aliases:
    - /2017/09/windows-mongodb-replica-set-auth.html
---
# 在 Windows 啟用 MongoDB Replica Set 驗證
之前曾在 [為 MongoDB 加上驗證機制](/2017/08/mongodb-enable-auth.html) 介紹過如何為 MongoDB 建立驗證授權機制，為資料加上基本保護，也在 [MongoDB 在 Windows 上的 HA 機制 - Replica sets](/2017/09/mongodb-windows-ha-replica-sets.html) 介紹 MongoDB 的 HA 機制 - Replica Set 相關安裝及設定。當然想要將 MongoDB 實際使用在 production 環境上，就必需兼顧資安及 HA 機制

加上近日 MongoDB 資安問題又再次佔據新聞版面 (詳細內容可以參考 [新一波MongoDB勒贖攻擊來襲，2.6萬台伺服器受害](http://www.ithome.com.tw/news/116651))，雖然數量下降，但攻擊強度卻有過之而無不及，所以今天就來紀錄如何啟用 Windows 中 MongoDB Replica Set 的認證與授權

## Internal Authentication 使用兩種驗證機制

1.  Keyfiles
    *   使用 SCRAM-SHA-1 建立密碼檔
    *   密碼長度需要介於 6 - 1024 字元
    *   需在各個 replica set or sharded clusters 間使用相同 keyfiles

2.  x.509
    *   使用 TLS/SSL 連線
    *   不使用帳號及密碼驗證

## 啟用 Replica Set 驗證機制

1.  先完成 Replica Set 相關設定

    > 詳細內谷可以參考 [MongoDB 在 Windows 上的 HA 機制 - Replica sets](/2017/09/mongodb-windows-ha-replica-sets.html)

2.  關閉 MongoDB Replica Set 各個 instance

3.  建立 `keyfile`

    *   可以使用任意方式產生 6 至 1024 個字元的隨機字串當做 key
    *   官方範例中使用 `openssl` 來產生隨機字串(windows 環境中，安裝 git 時會附帶安裝 `openssl.exe`)

        ```
        "C:\Program Files\Git\mingw64\bin\openssl.exe" rand -base64 741 > c:\keyfile
        ```

4.  將 `keyfile` 複製至各個 MongoDB server 上，並指定給各個 MongoDB instance

    *   在 `yaml` 設定檔中指定

        ```yml
        security:
            keyFile: c:\keyfile
        ```

    *   透過指令指定

        ```
        mongod --keyFile c:\keyfile
        ```
5.  重新啟動 MongoDB Replica Set 各個 instance

6.  設定第一個 user

    *   連線至 `primary`
    *   切換至 `admin` database

        ```
        use admin
        ```

    *   建立 `user` 並指定角色為 `userAdminAnyDatabase`

        ```
        db.createUser({user: "yowkoAdmin",pwd: "pass.123",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] })
        ```

        ![1createadmin](https://user-images.githubusercontent.com/3851540/30250172-6f633092-967c-11e7-9f68-fd6e7b544237.png)

    *   建立第一個 user 的動作需透過指令進行，無法使用 GUI 工具

        ![2connectfail](https://user-images.githubusercontent.com/3851540/30250173-6f6a1042-967c-11e7-85e7-a328ce5898a4.png)

    *   第一個 user 建立後即可使用 GUI 工具，指定 authentication 登入

        ![3connectpass](https://user-images.githubusercontent.com/3851540/30250166-6f1cbcfc-967c-11e7-8f0b-e60322706286.png)

7.  連線至 MongoDB
    *   使用指令
        *   語法

            ```
            mongo --port {port} -u "{帳號}" -p "{密碼}" --authenticationDatabase "{database}"
            ```

        *   範例

            ```
            mongo --port 27017 -u "yowkoAdmin" -p "pass.123" --authenticationDatabase "admin"
            ```

    *   使用 GUI
        *   加入連線資訊

            ![4addconnect1](https://user-images.githubusercontent.com/3851540/30250174-6f6ba4e8-967c-11e7-9178-712e0f37be1a.png)

            ![5addconnect2](https://user-images.githubusercontent.com/3851540/30250167-6f1d0a9a-967c-11e7-80d9-b0f37ae0a86f.png)

        *   選擇連線類型 --> Replica Set 並設定目標 Server

            ![6addconnect3](https://user-images.githubusercontent.com/3851540/30250169-6f43538a-967c-11e7-91de-355c5f169702.png)

        *   設定連線帳號密碼

            ![7addconnect4](https://user-images.githubusercontent.com/3851540/30250168-6f4350ec-967c-11e7-9bf9-1a95d05b5062.png)

## 建立儲存資料的 DB 及對應帳號權限

> 上述建立的帳號是針對 MongoDB 的管理權限，實際針對不同 DB 需要加入不同使用者及相關權限，才能實際讀寫資料，帳號設定可以參考 [為 MongoDB 加上驗證機制](/2017/08/mongodb-enable-auth.html)

1.  未建立 db 使用者沒有權限

    ![8unauth](https://user-images.githubusercontent.com/3851540/30250170-6f5fd99c-967c-11e7-82a4-bbaf4b6be153.png)

2.  建立使用者

    ```
    use test
    db.createUser(
    {
        user: "yowko",
        pwd: "pass.123",
        roles: [ { role: "readWrite", db: "test" } ]
    }
    )
    ```

3.  使用正確使用者登入

    ```
    use test
    db.auth("yowko","pass.123")
    ```

4.  新增成功

    ```
    db.test.insert({"aaa":123})
    ```

    ![9insertok](https://user-images.githubusercontent.com/3851540/30250171-6f6016be-967c-11e7-9171-7955d2deb58a.png)

## 心得

雖說 MongoDB 的設計概念就是用於信任網路，但為什麼不能預設使用較高層級的安全性保護機制，如果真的用不到再自行移除就好，至少可以避免一些偷懶的 user 所造成的低級資安疑慮

回到 MongoDB Replica Set 的驗證設定，設定與 [為 MongoDB 加上驗證機制](/2017/08/mongodb-enable-auth.html) 相去不遠，概念也相同，再搭配 [MongoDB 在 Windows 上的 HA 機制 - Replica sets](/2017/09/mongodb-windows-ha-replica-sets.html) 設定 Replica Set ，使用上並不複雜，但在釐清設定的過程中，花了不少時間驗證正確的做法，但終於對相關設定有比較明確的認識

# 參考資訊

1.  [為 MongoDB 加上驗證機制](/2017/08/mongodb-enable-auth.html)
2.  [MongoDB 在 Windows 上的 HA 機制 - Replica sets](/2017/09/mongodb-windows-ha-replica-sets.html)
3.  [新一波MongoDB勒贖攻擊來襲，2.6萬台伺服器受害](http://www.ithome.com.tw/news/116651)
4.  [Deploy Replica Set and Configure Authentication and Authorization](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/)
5.  [Authentication](https://docs.mongodb.com/manual/core/authentication/)
