---
title: "如何避免多個 EntityFramework 6 instance 造成資料覆蓋問題 (DB First - Oracle)"
date: 2018-06-05T00:50:00+08:00
lastmod: 2021-11-03T00:50:20+08:00
draft: false
tags: ["ASP.NET MVC","Entity Framework","Oracle"]
slug: "entityframework-6-concurrency-oracle"
aliases:
    - /2018/06/entityframework-6-concurrency-oracle.html
---
## 如何避免多個 Entity Framework 6 instance 造成資料覆蓋問題 (DB First - Oracle)

之前筆記 [如何避免多個 Entity Framework 6 instance 造成資料覆蓋問題 (DB First - SQL Server)](/entityframework-6-concurrency) 提到 Entity Framework 使用上的限制，也紀錄如何透過在 SQL Server 的 table 上加入 rowversion 的欄位並將 Entity Framework 上該欄位的 Concurrency Mode 改為 fixed 就可以避免修改資料後遭其他人覆寫的狀況，那相同問題當然也會發生在 Oracle 上，就來看看 Oracle 該如何解決吧

## 重現問題

1. user A 開啟特定一筆資料 (name 為 Yowko) 想要修改內容
2. user B 也想修改同一筆資料 (name 為 Yowko)
3. 修改情境 (同時開啟資料，僅存檔順序不同)
    - user B 將 salary 改為 `100` 並存檔，user A 則改為 `110` 並存檔

        ![1b100a110](https://user-images.githubusercontent.com/3851540/40586516-9184cace-61f5-11e8-9a49-ecb28d37489c.png)
    - user A 將 salary 改為 `110` 並存檔，user B 則改為 `100`  並存檔

        ![2a1101b100](https://user-images.githubusercontent.com/3851540/40586518-91aad174-61f5-11e8-8e36-f56c3aabc964.png)
    - user B 將 salary 改為 `100` 並存檔，user A 則修改 name 為 `Yowko Tsai` 並存檔

        ![3b100aname](https://user-images.githubusercontent.com/3851540/40586519-91d5b33a-61f5-11e8-8530-dcc5ee814d6f.png)
    - user A 將 name 為 `Yowko Tsai` 並存檔，user B 則修改 salary 為 `100` 並存檔

        ![4anameb100](https://user-images.githubusercontent.com/3851540/40586520-91fd8d60-61f5-11e8-89e7-566262b6a6df.png)
4. 結果為何？
    - user B 將 salary 改為 100 並存檔，user A 則改為 110 並存檔

        ![5a110](https://user-images.githubusercontent.com/3851540/40586507-9011f1b2-61f5-11e8-8893-db12432fb3bd.png)
    - user A 將 salary 改為 110 並存檔，user B 則改為 100  並存檔

        ![6b100](https://user-images.githubusercontent.com/3851540/40586508-90393a88-61f5-11e8-855d-a16ab36b3d63.png)
    - user B 將 salary 改為 100 並存檔，user A 則修改 name 為 Yowko Tsai 並存檔

        ![7b100aname](https://user-images.githubusercontent.com/3851540/40586509-905f85bc-61f5-11e8-821f-734cf06a2999.png)
    - user A 將 name 為 Yowko Tsai 並存檔，user B 則修改 salary 為 100 並存檔

        ![8anameb100](https://user-images.githubusercontent.com/3851540/40586510-908596b2-61f5-11e8-9ebe-7fbd5dee44c6.png)

看完上面幾種情境真實操作的結果，我想應該沒有人可以接受，所以接著就來看看該如何設定

## 設定步驟

1. DB 相關設定
    - 建立 table

        > 加入  `ROWVERSION`

        ```sql
        CREATE TABLE "TEST"."USERS" 
        (
            "ID" NUMBER(10,0) NOT NULL ENABLE, 
            "NAME" VARCHAR2(50 BYTE), 
            "SALARY" NUMBER(10,0), 
            "ROWVERSION" NUMBER(*,0) NOT NULL ENABLE,  
            CONSTRAINT "ID_PK" PRIMARY KEY ("ID")
        );
        ```

    - 建立 `SEQUENCE` for `ID` 跟 `ROWVERSION`

        > 似 SQL Server 自動序號

        - ID

            ```sql
            CREATE SEQUENCE "TEST"."SQ_USERSID"  MINVALUE 1 MAXVALUE 9999999999 INCREMENT BY 1 START WITH 1 CACHE 20 NOORDER  NOCYCLE ;
            ```

        - ROWVERSION

            ```sql
            CREATE SEQUENCE "TEST"."SQ_USERSROWVERSION"  MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 1 CACHE 20 NOORDER  NOCYCLE ;
            ```

    - 建立 trigger for `ID` 跟 `ROWVERSION`
        - ID

            > 使用 trigger 在 insert 資料時取得自動序號填入

            ```sql
            create or replace trigger TEST.TR_USERSID
                before insert on TEST.USERS
                for each row
            begin
                select TEST.SQ_USERSID.nextval into :new.Id from dual;
            end;
            ```

        - ROWVERSION

            > 使用 trigger 在 insert 及 update 資料時取得自動序號修改值

            ```sql
            create or replace trigger TEST.TR_USERSROWVERSION
                before insert or update on TEST.USERS
                for each row
            begin
                :NEW.ROWVERSION := TEST.SQ_USERSROWVERSION.NEXTVAL;
            end;
            ```  

    - 為什麼不使用 `ORA_ROWSCN`
        - 建立 table 時加上 `ROWDEPENDENCIES`

            > 10 g 開始可以使用 `ORA_ROWSCN` ，但需要特別注意的是 `ORA_ROWSCN` 預設是透過 block 來追蹤變更，不是絕對的精準，可以在建立 table 時加上 `ROWDEPENDENCIES` 調整為追蹤 row

            ```sql
            CREATE TABLE "TEST"."USERS" 
            (
                "ID" NUMBER(10,0) NOT NULL ENABLE, 
                "NAME" VARCHAR2(50 BYTE), 
                "SALARY" NUMBER(10,0),
                CONSTRAINT "ID_PK" PRIMARY KEY ("ID")
            )
            ROWDEPENDENCIES 
            ;
            ```

            > 11 g 尚未支援 alter 方式改追蹤 row
        - `ORA_ROWSCN` 並非欄位屬性，需要直接從 script 動手才能取得

            > 與 Entity Framework db first 不好整合，需要手動加入 `DefiningQuery` 與 `ModificationFunctionMapping`，使用上討不到便利

            ```sql
            SELECT
              ORA_ROWSCN as "rowversion",
              "USERS"."ID" AS "ID",
              "USERS"."NAME" AS "NAME",
              "USERS"."SALARY" AS "SALARY"
              FROM "TEST"."USERS" "USERS"
            ```

2. Entity Framework 設定
    - 將 table 加入 edmx 中

        ![10updatefromdb](https://user-images.githubusercontent.com/3851540/40586512-90dae464-61f5-11e8-9166-663a30e39939.png)
    - 將 `ROWVERSION` 欄位 Concurrency Model 屬性改為 `Fixed`

        ![2concurrencyfixed](https://user-images.githubusercontent.com/3851540/40929945-ceccd2e6-6858-11e8-858a-103ec03d27c6.png)
3. 調整 View (Edit)

    > 讓 `ROWVERSION` 成為隱藏欄位

    ```cs
    @Html.HiddenFor(model=>model.ROWVERSION)
    ```

4. 針對資料過時錯誤做特別處理

    ```cs
    try
    {
        await db.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException ex)
    {
        return Content($"<script>alert('資料已被他人修改，請重新編輯');document.location = '{Url.Action("Edit",user.ID)}'</script>");
    }
    ```

5. 實際效果

    ![12result](https://user-images.githubusercontent.com/3851540/40586514-91304c2e-61f5-11e8-88f7-355e254289c3.png)

## 心得

原本興高采烈以為 Oracle 也有跟 SQL Server 一樣的 rowversion 欄位屬性，可以延續過去處理並行衝突的經驗：設定 Concurrency 為 Fixed 即可，但實作後發現兩者終究是不同的設計概念， Oracle 不僅預設是追蹤 block 的變更再加上並非是欄位屬性，讓實際與 Entity Framework 整合時便利性大打折扣 (需要手動修改 edmx 的 `DefiningQuery` 與 `ModificationFunctionMapping` 區段，另外還需要有相關 Insert、Update、Delete DB function or storedprocedure 才能真正達成目的)，所以經過許多嘗試後還是決定放棄使用 `ORA_ROWSCN`，改透過 trigger 來改變 `ROWVERSION`的值，雖然看似沒那麼漂亮，不過使用起來反而比 `ORA_ROWSCN` 簡單許多XD

## 參考資訊

1. [ORA_ROWSCN Pseudocolumn](https://docs.oracle.com/cd/B28359_01/server.111/b28286/pseudocolumns007.htm)
2. [racle equivalent for SQL Server TIMESTAMP/ROWVERSION?](https://www.sqlservercentral.com/Forums/632973/Oracle-equivalent-for-SQL-Server-TIMESTAMPROWVERSION)
