---
title: "SQL Server - 無法建立 Database Diagrams"
date: 2017-10-11T22:16:00+08:00
lastmod: 2018-09-27T22:14:54+08:00
draft: false
tags: ["SQL Server"]
slug: "database-diagrams-backend-version"
aliases:
    - /2017/10/database-diagrams-backend-version.html
---
# SQL Server - 無法建立 Database Diagrams
想要了解 table 間的關聯與 table 的欄位屬性相關資訊，透過 SQL Server 的 Database Diagrams 是最直接方便的，所以新專案的部份功能想透過 Database Diagrams 來釐清 table 間相對關係，結果馬上就碰壁了，就來看看問題發生原因及解決方式吧

## 錯誤訊息

1.  訊息內容

    ```
    TITLE: Microsoft SQL Server Management Studio
    ------------------------------
            
    This backend version is not supported to design database diagrams or tables. (MS Visual Database Tools)

    ------------------------------
    BUTTONS:
    
    OK
    ------------------------------
    ```

2.  訊息截圖

    ![1error](https://user-images.githubusercontent.com/3851540/31445507-c49553a8-aed0-11e7-9d5f-915863fd1c3a.png)

## 問題發生原因

**SQL Server 的版本 比 SSMS 新**

*   SQL Server 版本

    > 執行 `SELECT @@VERSION`

    ![2serverversion](https://user-images.githubusercontent.com/3851540/31445508-c4bd5bd2-aed0-11e7-95aa-d0cf7c600ebb.png)

*   SSMS 版本

    > SSMS 主選單 Help --> About

    ![3helpabout](https://user-images.githubusercontent.com/3851540/31445509-c4e3f3dc-aed0-11e7-8594-2aa7b34e4a80.png)

    ![3ssmsversion](https://user-images.githubusercontent.com/3851540/31445510-c50ba346-aed0-11e7-8504-8d199a1f7001.png)

## 解決方式

**使用新版 SSMS**

![4newssms](https://user-images.githubusercontent.com/3851540/31445511-c5402008-aed0-11e7-88ac-46c144965e05.png)

![5ok](https://user-images.githubusercontent.com/3851540/31445512-c56afc6a-aed0-11e7-84e9-53eefa61d3de.png)

## 心得

無法建立 Database Diagrams 後，我才發現怎麼 localDB 被升級成 13 版，之前透過 SSMS 連線 `(localDB)\MSSQLLocalDB` 是連線至 12 版，測試資料庫也都是透過這個方式所建立的。

以現況來看，推測應該是安裝 Visual Studio 2017 時連帶升級所造成的，但不知道為什麼這次會直接升級既有資料庫而不是採用像 `(locadb)\v11.0` 跟 `(localDB)\MSSQLLocalDB` 的做法，另建安裝並保留舊有資料庫

# 參考資訊

1.  [The backend version is not supported to design database diagrams or tables](https://stackoverflow.com/questions/25146474/the-backend-version-is-not-supported-to-design-database-diagrams-or-tables)
