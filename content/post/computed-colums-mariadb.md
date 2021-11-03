---
title: "Generated Columns - MariaDB 上的 Computed Columns"
date: 2019-05-19T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["MariaDB"]
slug: "computed-colums-mariadb"
---
## Generated Columns - MariaDB 上的 Computed Columns

之前有個需求會用到合併既有的兩個欄位來進行資料檢索，當下想起了 SQL Server 上的 Computed Columns (計算欄位)，雖然後來採用了其他做法，不過既然都查了資料就紀錄一下吧

Generated Columns 是 MariaDB 10.2 加入的，語法上與 MySQL's generated columns 相同，包含兩種格式

1. `PERSISTENT` OR `STORED`

    > 會將值實際儲存在 table 中

2. `VIRTUAL`：預設值

    > 不會實際儲存在 table 中，會在 query 當下動態產生值

## 使用方式

1. 語法

    ```txt
    <type>  [GENERATED ALWAYS]  AS   ( <expression> )
    [VIRTUAL | PERSISTENT | STORED]  [UNIQUE] [UNIQUE KEY] [COMMENT <text>]
    ```

2. 範例

    ```sql
    create table DemoUser
    (
        FirstName varchar(5) null,
        SecondName varchar(15) null,
        FullName varchar(20) generated always AS (CONCAT(FirstName,' ',SecondName)) PERSISTENT
    );
    ```

3. 實際使用

    ```sql
    INSERT DemoUser Values('Yowko','Tsai',DEFAULT);

        select * from DemoUser;

    ```

    ![1output](https://user-images.githubusercontent.com/3851540/57984534-36cfa280-7a8f-11e9-92d0-35ddb3b7e172.png)

## 心得

雖然過去沒有使用 MariaDB 的經驗，所幸其他 DB 的經驗還是有些幫助，也許名稱不同但相同概念還是可以套用，這在其他技術面向上也適用：你覺得你學的某個東西、做的某個系統不是主流，過陣子你發現這就是你贏過別人的地方，你不再只是旁觀者，因為你接觸過所以你知道其中的眉眉角角

## 參考資訊

1. [Generated (Virtual and Persistent/Stored) Columns](https://mariadb.com/kb/en/library/generated-columns/)
