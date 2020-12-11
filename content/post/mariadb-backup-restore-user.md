---
title: "建立 MariaDB 備份與還原帳號"
date: 2020-11-21T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["MariaDB"]
slug: "mariadb-backup-restore-user"
---

## 建立 MariaDB 備份與還原帳號

之前筆記 [建立 MongoDB 自訂角色 (role)](/mongodb-custom-role/) 紀錄到如何在 MongoDB 上建立用來進行備份與還原的角色，再依這個角色建立使用者，相同的需求在 MariaDB 上也有，只是做法些許不同，紀錄一下備忘

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop community 2.5.0.1(49550)
3. docker images
    - mariadb 10.5.8

## 設定語法與說明

- 建立 account 並指定密碼

    - 語法

        ```bash
        CREATE USER '{username}'[@'{host}'] IDENTIFIED BY '{password}';
        ```

    - 範例

        ```bash
        CREATE USER 'backupuser' IDENTIFIED BY 'pass.123';
        ```

        - `host` 預設為 `%`
        - `host` 若為 `localhost` 僅允許 local 登入

- 備份權限

    1. 備份 table 權限
       
        - 語法
    
            ```bash
            GRANT SELECT, LOCK TABLES on {db name}.* TO '{username}';
            ```

        - 範例

            ```bash
            GRANT SELECT, LOCK TABLES on test.* TO 'backupuser';
            ```

            - 表示該帳號可以備份 `test` db 下所有 table
            - 可以使用 `*.*` 表示所有 db 的所有 table

    2. 備份 stored procedure 或是 functions 權限

        > 這個權限沒給，在執行備份時不會出現錯誤，但 stored procedure 與 functions 將直接被忽略

        - 語法
        
            ```bash
            GRANT SELECT on mysql.proc TO '{username}';
            ```
        
        - 範例

            ```bash
            GRANT SELECT on mysql.proc TO 'backupuser';
            ```

    3. 備份語法

        - 語法

            ```bash
            mysqldump -u {username} -p{password} --routines --compress --column-statistics=0 {db name}  > {backup target file name}
            ```

        - 範例

            ```bash
            mysqldump -u backupuser -ppass.123 --routines --compress --column-statistics=0 test > test.sql
            ```

- 還原權限

    1. grant 修改斷行符號 (DELIMITER) 權限

        > `SUPER` 的 `GLOBAL` 層級的權限設定

        - 語法

            ```bash
            GRANT SUPER on *.* to '{username}';
            ```

        - 範例

            ```bash
            GRANT SUPER on *.* to 'backupuser';
            ```
    
    2. grant 還原 stored procedure、function 與 table schema、table data 權限

        - 語法

            ```bash
            GRANT ALTER ROUTINE,CREATE ROUTINE,SELECT, INSERT, LOCK TABLES, CREATE, Drop, CREATE TEMPORARY TABLES, INDEX, ALTER on {db name}.* TO '{username}';
            ```

        - 範例

            ```bash
            GRANT ALTER ROUTINE,CREATE ROUTINE,SELECT, INSERT, LOCK TABLES, CREATE, Drop, CREATE TEMPORARY TABLES, INDEX, ALTER on test.* TO 'backupuser';
            ```

    3. 還原語法

        - 語法

            ```bash
             mysql -u{username} -p{password} {db name} < {target restore sql file name}
            ```

        - 範例

            ```bash
             mysql-ubackupuser -ppass.123 test <test.sql
            ```

## 心得

不知道是 mariadb 資源比較少，還是大家在帳號的使用權限控管上比較沒有拆分那麼細，網路上的資料不是很好找，[MySQL – MariaDB – User Accounts and Privileges](https://devopspoints.com/mysql-mariadb-user-accounts-and-privileges.html) 已經算是相當完整的，只是在 stored procedure、function 的處理還有遺漏

幸虧在測試數十次後，勉強找出可用的版本，但不是 dba 說實話也不知道權限使用上是不是正確，就先紀錄一下，日後被指正再調整囉

## 參考資訊

1. [建立 MongoDB 自訂角色 (role)](/mongodb-custom-role/)
2. [Host Name Component](https://mariadb.com/kb/en/create-user/#host-name-component)
3. [SUPER](https://mariadb.com/kb/en/grant/#super)
4. [mysqldump not dumping stored procedures](https://stackoverflow.com/a/46556708)
5. [mysqldump throws: Unknown table 'COLUMN_STATISTICS' in information_schema (1109)](https://serverfault.com/questions/912162/mysqldump-throws-unknown-table-column-statistics-in-information-schema-1109)
6. [MySQL – MariaDB – User Accounts and Privileges](https://devopspoints.com/mysql-mariadb-user-accounts-and-privileges.html)
