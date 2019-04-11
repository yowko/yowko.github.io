---
title: "Oracle 如何做到 SQL Server 的 Identity 欄位型態"
date: 2017-10-28T15:41:00+08:00
lastmod: 2018-09-27T15:41:16+08:00
draft: false
tags: ["Oracle"]
slug: "oracle-identity"
aliases:
    - /2017/10/oracle-identity.html
---
# Oracle 如何做到 SQL Server 的 Identity 欄位型態
在 DB table 規劃時，內部使用的 primary key 欄位常見用法分成兩派：使用 int 或是 GUID，在 SQL Server 上兩種做法都有對應的欄位型態，而在 Oracle 上就得要自行處理了，以 int 做法為例，在 SQL Server 中實作的欄位型態為 `Identity`，是一種自動累加固定值的識別碼，馬上來看看該如何在 Oracle 上實作 Identity 囉

## 實作做法說明

1.  依實際需求建立 table 及欄位，並指定 primary key 欄位為 number 型態
2.  建立 oracle sequence (序列) 用來取號
3.  建立 trigger 在 insert data 時從 sequence 取號塞值


## 1. 建立 table

> 請依實際需求調整 table 欄位

*   需要 primary key 為 `number(10, 0)` 且 `not null`

    > 我這邊使用 `Id` 為欄位名稱

*   指定 constraint
*   語法範例

    ```sql
    create table UserProfile
    (
        Id number(10, 0) not null, 
        Name VARCHAR2(50) not null, 
        BirthDay date not null, 
        Addr VARCHAR2(250) not null,
        constraint PK_UserProfile primary key (Id)
    )
    ```

## 2. 建立 Oracle Sequence

*   用來自動循序增加數字
*   語法範例

    ```sql
    create sequence SQ_UserProfile
    ```

## 3. 建立 Trigger

*   在 insert 資料前到 sequence 取號寫入 primary key 欄位
*   語法範例

    ```sql
    create or replace trigger TR_UserProfile
    before insert on UserProfile
    for each row
    begin
    select SQ_UserProfile.nextval into :new.Id from dual;
    end;
    ```

## 實際效果

*   輸入時指定 `100`

    ![1input](https://user-images.githubusercontent.com/3851540/32132311-64a94af4-bbf4-11e7-8a12-9892b83fb57b.png)

*   儲存 `1`

    ![2save](https://user-images.githubusercontent.com/3851540/32132312-64dc3892-bbf4-11e7-8a3c-1320feaf18eb.png)

## 心得

一直以來都是使用 SQL Server 從來沒想過原來其他 db 沒有 identity 欄位型態，本來也沒想到可以這麼做，是使用 code first 產生 pl-sql 時發現原來 oracle 的 identity 可以透過上述方式建立，特別筆記一下

# 參考資訊
