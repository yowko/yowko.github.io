---
title: "SQL Server 無法新增資料庫圖表(database diagram)"
date: 2017-04-06T23:23:00+08:00
lastmod: 2021-10-26T23:23:09+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-database-diagram"
aliases:
    - /2017/04/sql-server-database-diagram.html
---
## QL Server 無法新增資料庫圖表(database diagram)

因為之前工作常需要建立新的資料庫及資料表，在保哥指導下學會使用以資料庫圖表(database diagram)來快速建立資料表的欄位內容及關聯，讓整個資料表的關係與欄位一目了然，是個很棒的方法，有時專案需要附上 db 相關說明，更是好用的不得了

最近因為專案需要又重新開始利用資料庫圖表(database diagram)來加速開發，但因為無有效擁有者而無法建立資料庫圖表(database diagram)的問題已經遇到兩次，所以筆記一下該如何解決

## 錯誤訊息

1. 訊息內容

    ```log
    TITLE: Microsoft SQL Server Management Studio
    ------------------------------

    Database diagram support objects cannot be installed because this database does not have a valid owner.  To continue, first use the Files page of the Database Properties dialog box or the ALTER AUTHORIZATION statement to set the database owner to a valid login, then add the database diagram support objects.

    ------------------------------
    BUTTONS:

    OK
    ------------------------------
    ```

2. 錯誤截圖

    ![1errormessage](https://cloud.githubusercontent.com/assets/3851540/24761793/076cbe5a-1b1f-11e7-856c-d00871ab0fec.png)

## 如何排除問題(擇一即可)

* 使用 UI 設定
    1. database 上按右鍵 --> Properties

        ![2properties](https://cloud.githubusercontent.com/assets/3851540/24761794/07ac33fa-1b1f-11e7-99af-aef6ab1cfcf1.png)

    2. Files --> Select Owner

        ![3selectowner](https://cloud.githubusercontent.com/assets/3851540/24761799/07befaa8-1b1f-11e7-9c8d-887acd2ca9b3.png)

    3. 選擇適合的 object name

        ![4bowser](https://cloud.githubusercontent.com/assets/3851540/24761800/07c0b01e-1b1f-11e7-8af2-472267f94545.png)

* 使用 script
    `ALTER AUTHORIZATION ON DATABASE::{database} TO [{username}];`

  * e.g.

    > ALTER AUTHORIZATION ON DATABASE::AdventureWorks2014 TO [sa];

## 設定完成

1. database 的 Database diagrams 上右鍵 --> New Database diagram

    ![5newdiagram](https://cloud.githubusercontent.com/assets/3851540/24761797/07bc35a2-1b1f-11e7-8078-a02d2b844059.png)

2. 錯誤訊息已消失

    ![6creatediagram](https://cloud.githubusercontent.com/assets/3851540/24761798/07bf1c86-1b1f-11e7-8b61-45fd7ca04c66.png)

## 參考資訊

1. [Database Diagram Support Objects cannot be Installed … no valid owner](http://stackoverflow.com/questions/2043382/database-diagram-support-objects-cannot-be-installed-no-valid-owner)
