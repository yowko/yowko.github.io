---
title: "為 MongoDB 加上驗證機制"
date: 2017-08-29T00:54:00+08:00
lastmod: 2021-11-02T13:56:11+08:00
draft: false
tags: ["MongoDB"]
slug: "mongodb-enable-auth"
aliases:
    - /2017/08/mongodb-enable-auth.html
    - /2017/august/mongodb-enable-auth
---
## 為 MongoDB 加上驗證機制

資訊安全總是資訊相關工作者的罩門之一：功能愈加愈多、時程愈縮愈短、製作費用愈來愈少，而要求卻愈來愈難，常常為了滿足合約上明列的基本功能就捉襟見肘，導致其他更重要但未列在合約上的細節，就這麼被忽略了，資安就是其中一項

一兩年前，資訊界傳出幾個大型勒索案一反傳統透過 sql-injection 洩露資訊的管道，有心人士搭著 nosql 的堀起，看準大家在應用 nosql 時為求方便常常使用預設設定，而 nosql 的預設設定普遍沒有驗證保護，造成大量機敏資料在網路上裸奔，其中一個常見的受害者就是 MongoDB

為了避免所使用的 MongoDB 資料赤赤裸裸地攤在陽光下，我們一定要學會如何為 MongoDB 加上基本驗證功能

## 設定驗證機制

1. 必需先建立 MongoDB 的管理者才會啟動驗證機制
2. 為特定 db 加上使用者及帳密保護

## 建立 MongoDB 的管理者

1. 啟動 MongoDB (已啟動可忽略)

    > 之前筆記 [如何在 Windows 環境安裝及設定 MongoDB](/2017/08/windows-mongodb.html) 有詳細介紹，這邊不重複贅述

2. 連線至 MongoDB

    ```cmd
    mongo --host {serverIP} --port {port}
    ```

    ![1mongoconnect](https://user-images.githubusercontent.com/3851540/29783678-efa1eb0a-8c53-11e7-9d20-1582c3b91500.png)

3. 切換至 `admin` 資料庫

    ```cmd
    use admin
    ```

    ![2useadmin](https://user-images.githubusercontent.com/3851540/29783679-efc6085a-8c53-11e7-9033-49be5f5563f1.png)

4. 加入帳號密碼並設定 role

    ```cmd
    db.createUser(
    {
        user: "yowko",
        pwd: "password",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]// role 只接受 `userAdmin` 與 `userAdminAnyDatabase`
    })
    ```

    > role 只接受 `userAdmin` 與 `userAdminAnyDatabase`

    ![3adduser](https://user-images.githubusercontent.com/3851540/29783680-efd14666-8c53-11e7-9a07-2430d814ffa5.png)

5. 重新啟動 MongoDB (擇一即可)

    * 使用指令需加上 `--auth`

        ```cmd
        mongod.exe --auth --config db1.conf
        ```

    * 調整 config

        ```yml
        security:
            authorization: enabled
        ```

6. 使用帳密登入 MongoDB

    * 方法一：連線時直接指定帳號密碼

        ```cmd
        mongo --host {serverIP} --port {port} -u "{username}" -p "{password}" --authenticationDatabase "admin"
        ```

        ![4connectwithauth](https://user-images.githubusercontent.com/3851540/29783682-efd647c4-8c53-11e7-9ed4-f7b2885dfbe4.png)

    * 方法二：連線後再使用帳號密碼

        ```cmd
        mongo --host {serverIP} --port {port}
        use admin
        db.auth("{username}","password")
        ```

        ![5connectauth](https://user-images.githubusercontent.com/3851540/29783681-efd38b6a-8c53-11e7-91cf-0f8d2a9cc3c6.png)

## 加入一般 DB 使用者

1. 切換至目標 DB 並建立使用者

    ```cmd
    use test
    db.createUser(
    {
        user: "yowkoTest",
        pwd: "passwordTest",
        roles: [ { role: "readWrite", db: "test" } ]
    }
    )
    ```

    ![7createuser](https://user-images.githubusercontent.com/3851540/29783676-ef9b3b02-8c53-11e7-8d48-3dbe72fe9ba0.png)

2. 連線至目標 db

    > 連線方式與上述相同

    * 方法一：連線時直接指定帳號密碼

        ```cmd
        mongo --host {serverIP} --port {port} -u "{username}" -p "{password}" --authenticationDatabase "test"
        ```

    * 方法二：連線後再使用帳號密碼

        ```cmd
        mongo --host {serverIP} --port {port}
        use test
        db.auth("{username}","password")
        ```

3. insert 資料

    ```js
    db.test.insert({x:1,y:1})
    ```

    ![8insert](https://user-images.githubusercontent.com/3851540/29783677-ef9ec92a-8c53-11e7-8d49-57b643236bb2.png)

## 心得

設定方式並不困難，只是預設不啟用，而造成不少機敏資訊外洩的風險。

有時可能會覺得放在內網很安全，不會被攻擊，但資料不一定是被攻擊才會消失，有時只是因為操作失誤就可能造成資料被清除，所以一定要記得為資料做些基本保護

## 參考資訊

1. [Enable Auth](https://docs.mongodb.com/manual/tutorial/enable-authentication/)
2. [如何在 Windows 環境安裝及設定 MongoDB](/2017/08/windows-mongodb.html)
