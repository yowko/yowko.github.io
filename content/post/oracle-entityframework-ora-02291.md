---
title: "Oracle 搭配 Entity Framework 寫入資料出現 ORA-02291 錯誤"
date: 2017-10-29T01:31:00+08:00
lastmod: 2020-12-11T01:31:37+08:00
draft: false
tags: ["Debug","Entity Framework","Oracle"]
slug: "oracle-entityframework-ora-02291"
aliases:
    - /2017/10/oracle-entityframework-ora-02291.html
---
# Oracle 搭配 Entity Framework 寫入資料出現 ORA-02291 錯誤
之前文章 [Oracle 如何做到 SQL Server 的 Identity 欄位型態](/2017/10/oracle-identity.html) 介紹到在 Oracle 將 table 欄位設定成模擬 SQL Server Identity 的做法，但實際使用上卻發現某些 table 寫入時卻出現 `ORA-02291: integrity constraint (******.FK_****) violated - parent key not found` 錯誤，檢查了不少地方，最後將 SQL statement output 出來直接執行卻未發生錯誤，直覺與 Entity Framework 有關，接著就來看看如何解決與問題發生原因吧

## 錯誤訊息

*   訊息內容

    ```
    Server Error in '/' Application.

    ORA-02291: integrity constraint (TESTIDENTITY.FK_USERPROFILELOG) violated - parent key not found

    Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 

    Exception Details: Oracle.ManagedDataAccess.Client.OracleException: ORA-02291: integrity constraint (TESTIDENTITY.FK_USERPROFILELOG) violated - parent key not found

    Source Error: 


    Line 54:                 db.USERPROFILE.Add(uSERPROFILE);
    Line 55:                 uSERPROFILE.USERPROFILELOG.Add(new USERPROFILELOG() {LOGON =DateTime.Now});
    Line 56:                 await db.SaveChangesAsync();
    Line 57:                 return RedirectToAction("Index");
    Line 58:             }

    Source File: C:\Users\YowkoTsai\Documents\Visual Studio 2017\Projects\TestPermission\TestIdentity\Controllers\UserProfileController.cs    Line: 56 
    ```
*   錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/32136830-e563d8aa-bc47-11e7-8009-bd7e506dcaf6.png)

## 情境說明

1.  使用 Entity Framework 連線至 Oracle
2.  多個 table 間互有關聯

    ![2tablerela](https://user-images.githubusercontent.com/3851540/32136831-e58bc9aa-bc47-11e7-95ee-77b6fb4b7312.png)

3.  寫入資料內容涵蓋多個 table

    ![3subobject](https://user-images.githubusercontent.com/3851540/32136832-e5b38cec-bc47-11e7-90c5-cae9b74540c2.png)

## 解決方式

*   將 master table 的 primary key 在 Entity Framework 的對應方式改為 `Identity`

    ![4resolve](https://user-images.githubusercontent.com/3851540/32136833-e5db71a8-bc47-11e7-93f0-ff426b7167ab.png)

## 問題原因

1.  sub entity 要寫入 db 時需要使用 master table primary key 做為 foreign key，而傳入的 master entity primary key 與 sequence 產生序號不同，造成 foreign key constaint 檢查找不到資料
    *   實例
        *   傳入 object 如下

            ```
            masterentity
            {
                Id:12,
                subentity
                {
                    Id:999
                    MasterId:12
                }
            }
            ```

        *   情況一：sequence Id 12 不存在出現 ORA-02291 錯誤(本次錯誤)
        *   情況二：sequence Id 12 已建立過，造成 sub entity 對應錯誤(更慘，要特別注意)

    *   改為 `Identity` 後即標記讓欄位由 DB 處理

2.  單一資料表不會出現錯誤是因為不需檢查 foreign key constaint


## 心得

問題解決方式 google 後馬上就得到正確解答了，只是解決問題當下一時沒有意會過來真正原因，經過後來仔細回想才真正參透其中的關聯，筆記一下加深印象

# 參考資訊

1.  [Entity Framework with Oracle Insert many to many error](https://stackoverflow.com/questions/21843128/entityframework-with-oracle-insert-many-to-many-error)
