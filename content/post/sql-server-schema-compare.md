---
title: "如何快速比對資料庫結構差異並產生同步指令"
date: 2017-04-10T00:20:00+08:00
lastmod: 2021-10-15T00:20:54+08:00
draft: false
tags: ["SQL Server","Visual Studio"]
slug: "sql-server-schema-compare"
aliases:
    - /2017/04/sql-server-schema-compare.html
---
## 如何快速比對資料庫結構差異並產生同步指令

之前文章 [如何為 SQL SERVER 建立資料庫版控](//blog.yowko.com/2017/04/sql-server-data-tools.html) 簡短地介紹如何使用 SQL Server Data Tools (SSDT) 來將資料庫物件轉換為程式碼以利進行版控，接著要來介紹 SSDT 另個好用的功能：Schema 比對

資料庫版控對開發人員是一大福音，但如果只是知道版本差異而無法快速產生 sync db 的 script 對開發流程改善的效益還是有限，微軟 SSDT 開發人員也為我們設想到了：我們可以透過 SQL Server Data Tools (SSDT) 來快速比對資料庫間的差異，也可以比對資料庫及資料庫專案(轉換為程式碼的資料庫)，比較完成後不僅可以直接從 Visual Studio 異動資料庫，還能產生出 sync script 保留了讓 DBA 介入的彈性，讓無法由開發人員直接異動的環境也可以享受到便利性

## 新增 schema 比對

> 方式有兩個(擇一即可)

1. 專案右鍵 --> Schema Compare...

    ![1addcompare](https://cloud.githubusercontent.com/assets/3851540/24838792/97cbf416-1d81-11e7-928d-d06fb5bf38a0.png)

    * 透過這個方式建立 schema 比對，預設 source 就會指向資料庫專案

        ![3defaultproject](https://cloud.githubusercontent.com/assets/3851540/24838799/97f2735c-1d81-11e7-8e0e-844d5101933c.png)

2. Visual Studio 主選單中的 Tools --> SQL Server --> New Schema Comparison...

    ![2newcompare](https://cloud.githubusercontent.com/assets/3851540/24838790/97cbba78-1d81-11e7-8a51-518751835895.png)

    * 透過這個方式建立 schema 比對，source 與 target 都會是空的，比較適合用來做不同資料庫間的比對

        ![4emptysourcetarget](https://cloud.githubusercontent.com/assets/3851540/24838794/97f09bfe-1d81-11e7-97e4-8c59d136a7ec.png)

3. 選擇比對的 source 與 target

    > 以下以 target 為例(source 選擇方式也相同)

    * 3-1. Select Target

        ![5selecttarget](https://cloud.githubusercontent.com/assets/3851540/24838796/97f12998-1d81-11e7-9278-56ef7d17bf05.png)

    * 3-2. 支援三種比對類型

        ![6SCHEMATYPE](https://cloud.githubusercontent.com/assets/3851540/24838795/97f128da-1d81-11e7-9052-287184d45c3e.png)

        * Project

            > 資料庫專案

        * Database

            > 一般資料庫連線

        * Data-tier Application File

            > 使用 DAC 檔，這個另外擇日紀錄

    * 3-3. 選擇或是填寫資料連線資訊

        ![7connection](https://cloud.githubusercontent.com/assets/3851540/24838798/97f2a462-1d81-11e7-8361-28cc20d73b9a.png)

        ![8connection](https://cloud.githubusercontent.com/assets/3851540/24838797/97f277d0-1d81-11e7-811a-f01a935ea693.png)

## 執行 schema 比對 : project - database

<div style="color:red">需要特別注意 source 與 target 的位置</div>

為什麼需要特別強調 source 與 target 位置，因為位置一錯可能你好幾天辛苦的工作就會付之一炬了

* source 指的是 `更新的來源依據` (比較新的 schema)
* target 則是 `想要更新的目標` (比較舊的 schema)

1. 開始 Compare

    ![9COMPARE](https://cloud.githubusercontent.com/assets/3851540/24838800/9815c438-1d81-11e7-86db-00ef87e88f58.png)

2. 取得結果

    ![10result](https://cloud.githubusercontent.com/assets/3851540/24838801/98162af4-1d81-11e7-8a5e-05d0bc3007c5.png)

3. 更新資料庫

    ![11update](https://cloud.githubusercontent.com/assets/3851540/24838802/98177aa8-1d81-11e7-939c-cacf5ebffedc.png)

4. 產生資料庫 sync script

    ![12genscript](https://cloud.githubusercontent.com/assets/3851540/24838788/97ca2de8-1d81-11e7-9ecc-174939567eaa.png)

5. 確認 script 正確性

    ![12confirm](https://cloud.githubusercontent.com/assets/3851540/24838803/9818ded4-1d81-11e7-864e-09b6433e5453.png)

    * 以上述截圖例子就是有人直接在 db 中加了一個欄位，而沒有加入版控中，所以如果直接 update 或是 執行 script 就會刪除這個欄位，而如果是想要將這個欄位加入版控中，只要將 source 與 target 交換即可

        ![13switch](https://cloud.githubusercontent.com/assets/3851540/24838791/97cb9f52-1d81-11e7-8e9f-ad38eba56ca4.png)

        ![14schemaupdate](https://cloud.githubusercontent.com/assets/3851540/24838789/97ca7cda-1d81-11e7-8a6d-b85d56771793.png)

## 執行 schema 比對 : database - database

<div style="color:red">需要特別注意 source 與 target 的位置</div>

* 步驟與上述 project - database 皆相同，只是上述流程 source or target 有一個是資料庫專案 (project)，database to database 就是 source 與 target 各為兩個不同的 database

* 下列範例使用 sqlexpress 與 localdb 來模擬

    ![15db2db](https://cloud.githubusercontent.com/assets/3851540/24838793/97cc8d0e-1d81-11e7-8a4a-b7ba598fca89.png)

## 參考資訊

1. [如何為 SQL SERVER 建立資料庫版控](//blog.yowko.com/2017/04/sql-server-data-tools.html)
