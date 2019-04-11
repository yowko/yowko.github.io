---
title: "三種用來備份還原 SQL Server schema 與資料的做法"
date: 2017-04-16T23:23:00+08:00
lastmod: 2018-09-17T23:23:00+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-backup-restore"
aliases:
    - /2017/04/sql-server-backup-restore.html
---
# 三種用來備份還原 SQL Server schema 與資料的做法
部份公司團隊在專案開發流程上有較多的環境需要建置，或是有時需要針對不同環境的資料及 schema 進行除錯與比對時就是件辛苦的事了，這樣的工作我們一定需要透過工具來處理，避免人為的疏忽也讓人力去做更有價值的工作

但工具的安裝在許多環境是不被允許的，或是有些環境的權限比較嚴謹，我們的工具無法直接存取目接資料庫或是沒有目標資料庫的權限，這時我們就得考慮將資料庫備份回其他環境進行還原才能完成比對工作

所以來介紹工作中常使用的幾種 SQL Server 備份還原方式及優缺點

## 一、使用 .bak 檔案

1.  備份

    *   目標資料庫 --> 右鍵 --> Tasks --> Back Up..

        ![1backup](https://cloud.githubusercontent.com/assets/3851540/25072129/219ed91a-22fa-11e7-9e8a-e8a93d1d7e5d.png)

    *   備份選項

        *   主要是指定備份檔案名稱

            ![2options](https://cloud.githubusercontent.com/assets/3851540/25072127/219d180a-22fa-11e7-98e0-403e44cf608a.png)

    *   備份成功
        
        ![3success](https://cloud.githubusercontent.com/assets/3851540/25072128/219d2c32-22fa-11e7-88ee-a232824cc2cb.png)

2.  還原

    *   Databases 資料夾 --> 右鍵 --> Restore Database...
        
        ![4restore](https://cloud.githubusercontent.com/assets/3851540/25072131/21c26556-22fa-11e7-9f1e-575a7a5c3be9.png)

    *   Device --> ...

        ![5device](https://cloud.githubusercontent.com/assets/3851540/25072132/21c51b8e-22fa-11e7-9ce9-aa092d1a0e97.png)

    *   Select backup devices

        ![6selectdevice](https://cloud.githubusercontent.com/assets/3851540/25072133/21c6f13e-22fa-11e7-9625-3a963392ac00.png)

    *   選取後就會將相關資訊填入 Destination 及 Restore plan 中

        ![7restoreplan](https://cloud.githubusercontent.com/assets/3851540/25072134/21c7e4d6-22fa-11e7-8399-d8c026e2d416.png)

    *   其他還原選項:資料庫存在時的策略、資料庫檔案儲存位置

        ![8restoreoption](https://cloud.githubusercontent.com/assets/3851540/25072135/21cb3136-22fa-11e7-9951-2f306fca381d.png)

3.  優缺點


    *   優點：

         1.  備份還原速度快，三種方式最快的

    *   缺點：

        1.  新版 SQL Server 備份出來的檔案無法還原至較舊的 SQL Server 中
        2.  Azure SQL 無法使用

## 二、使用 script

1.  備份


    *  目標資料庫 --> 右鍵 --> Tasks --> Generate Scripts..

        ![9genscript](https://cloud.githubusercontent.com/assets/3851540/25072137/21ed0770-22fa-11e7-9837-a3fbbbc455cc.png)

    * Introduction

        ![10intro](https://cloud.githubusercontent.com/assets/3851540/25072136/21ecbe1e-22fa-11e7-9995-a42b800e43f6.png)

    *   Choose Onjects

        *   可以自由選擇需要匯出的資料庫物件


            ![11selectobject](https://cloud.githubusercontent.com/assets/3851540/25072139/21efb2b8-22fa-11e7-9a08-7cf5136375f8.png)

    * Set Scripting Options

        ![12scriptoption](https://cloud.githubusercontent.com/assets/3851540/25072138/21eea4c2-22fa-11e7-81de-e20beefc09df.png)

        *   紅框可以自訂進階匯出選項(以下分享幾個我自己常用的設定項目)


            *   Script DROP and  CREATE (選擇產生 drop 或 create 語法)

                ![13dropcreate](https://cloud.githubusercontent.com/assets/3851540/25072140/2201d164-22fa-11e7-9365-a827b67816af.png)

            *   Script Server Version (針對不同版本 SQL Server 產生對應的 script)

                ![14scriptversion](https://cloud.githubusercontent.com/assets/3851540/25072141/221314b0-22fa-11e7-9e28-6da8668b1072.png)

            *   Types of data to script (選擇產生 schema 或 data 的 script )

                ![15typescript](https://cloud.githubusercontent.com/assets/3851540/25072142/2216805a-22fa-11e7-8c61-5be46b1751a5.png)

            *   Script for the database engine type(選擇一般資料庫或是 Azure SQL)

                ![16enginetype](https://cloud.githubusercontent.com/assets/3851540/25072124/2146a4c0-22fa-11e7-90e8-4b833e23b658.png)

        *   綠框可以選擇匯出的目標
            *   儲存成檔案
            *   儲存至剪貼簿
            *   儲存至新的查詢視窗

    *  Summary

        ![17summary](https://cloud.githubusercontent.com/assets/3851540/25072125/2176ef22-22fa-11e7-9e1f-74298c7414f8.png)

    * Save of Publish Scripts

        ![18inprogress](https://cloud.githubusercontent.com/assets/3851540/25072126/219c191e-22fa-11e7-8c2a-454b5bf53868.png)

        ![19success](https://cloud.githubusercontent.com/assets/3851540/25072130/219f794c-22fa-11e7-83ef-81e875058669.png)

2.  還原

    > 直接執行產生的 script 即可

3.  優缺點
    *   優點：
        1.  可以選擇產出舊版 SQL Server 的 script
        2.  可以自行調整 script

    *   缺點：
        1. 特殊字元或是 binary 無法匯入
        2. 如果想要建立相同的資料庫需要調整的內容較多

## 三、使用 .bacpac

較詳細的內容可以參考 [你認識 SQL Server 的資料層應用程式(Data-tier Applications - DAC)嗎?](//blog.yowko.com/2017/04/sql-server-dac.html)

1.  備份
    *   目標資料庫 --> 右鍵 --> Tasks --> Export Data-tier Application...

        ![35export](https://cloud.githubusercontent.com/assets/3851540/25018469/ea8e9c86-20b9-11e7-9f19-63abe89eb86f.png)

    *   Introduction

        ![36intro](https://cloud.githubusercontent.com/assets/3851540/25018475/eab9f7a0-20b9-11e7-8668-e489065c49da.png)

    *   Export Settings

        *   Settings

            > 選擇儲存位置及檔名

            ![37settings](https://cloud.githubusercontent.com/assets/3851540/25018472/eaae4fa4-20b9-11e7-8e6c-e7d35c7f8a5d.png)

        * Advanced

            > 選擇匯出內容

            ![38advanced](https://cloud.githubusercontent.com/assets/3851540/25018473/eab5a588-20b9-11e7-803d-7f9571b9d624.png)

    * Summary

        ![39summary](https://cloud.githubusercontent.com/assets/3851540/25018474/eab81386-20b9-11e7-8d01-4ea2d525db65.png)

    * Results

        ![40inprogress](https://cloud.githubusercontent.com/assets/3851540/25018477/ead12b0a-20b9-11e7-8380-19fe6d4cf242.png)

        ![41success](https://cloud.githubusercontent.com/assets/3851540/25018476/eacbe2b2-20b9-11e7-8900-fa754148f4aa.png)

2.  還原


    * 資料庫資料夾 --> 右鍵 --> Import Data-tier Application...

        ![42import](https://cloud.githubusercontent.com/assets/3851540/25018478/eadf5798-20b9-11e7-9a3b-8675ae9c79c6.png)

    * Introduction

        ![43INTRO](https://cloud.githubusercontent.com/assets/3851540/25018433/e96db396-20b9-11e7-8a09-0d348c6d00e7.png)

    * Import Settings

        > 選擇 BACPAC 檔案位置

        ![44filelocation](https://cloud.githubusercontent.com/assets/3851540/25018430/e96c73be-20b9-11e7-8975-cd8bbd6759b4.png)

    * Database Settings

        > 設定 db 名稱及相關檔案儲放路徑

        ![45dbsetting](https://cloud.githubusercontent.com/assets/3851540/25018435/e9710190-20b9-11e7-857f-cebe749ed028.png)

    * Summary

        ![46summary](https://cloud.githubusercontent.com/assets/3851540/25018434/e96e2ee8-20b9-11e7-8894-97b8ec604436.png)

    * Results

        ![47inprogress](https://cloud.githubusercontent.com/assets/3851540/25018432/e96d40c8-20b9-11e7-932e-b5adafd63f1e.png)

        ![48success](https://cloud.githubusercontent.com/assets/3851540/25018431/e96ce9ac-20b9-11e7-8c52-c6fce60dab35.png)

3.  優缺點
    *   優點：
        1.  使用 XML 來進行資料描述，比 script 可攜性高
        2.  速度比產生 script 快
        3.  Azure SQL 可以直接使用

    *   缺點：
        1.  舊版 DAC 工具無法讀取新版 DAC 產生的檔案
        2.  有些比較進階的特性不支援

# 參考資訊
1.  [你認識 SQL Server 的資料層應用程式(Data-tier Applications - DAC)嗎?](//blog.yowko.com/2017/04/sql-server-dac.html)
