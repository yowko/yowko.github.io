---
title: "如何避免多個 Entity Framework 6 instance 造成資料覆蓋問題 (DB First - SQL Server)"
date: 2018-05-27T21:54:00+08:00
lastmod: 2018-10-06T21:54:55+08:00
draft: false
tags: ["ASP.NET MVC","EntityFramework","SQL Server"]
slug: "entityframework-6-concurrency"
aliases:
    - /2018/05/entityframework-6-concurrency.html
---
# 如何避免多個 Entity Framework 6 instance 造成資料覆蓋問題 (DB First - SQL Server)
前後台分離且共同存取 table 或是同時有多台機器甚至是修改資料不經由 Entity Framework 都是平常開發上很常見的情境，但這些操作卻可能因為 Entity Framework 的 cache 機制而出現資料不一致的現象，最近同事疑似遇到類似問題，雖然立馬就想到解決方式，但已經好一陣子沒用，為避免說錯就自己先測試一下並紀錄紀錄囉

要確保資料的完整及一致性，做法有好幾個包括 DB First、Code First、DB table setting 首先就從最簡單的做法：DB First + table setting + SQL Server 看起

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
1. DB Table 設定
    >加入 `rowversion` 欄位
    
    ![9addrowversion](https://user-images.githubusercontent.com/3851540/40586511-90ae1a2e-61f5-11e8-8fcc-4a11206053eb.png)

3. EntityFramework 設定
    - 更新 edmx 
        
        ![10updatefromdb](https://user-images.githubusercontent.com/3851540/40586512-90dae464-61f5-11e8-9166-663a30e39939.png)
    - 將 `rowversion` 欄位 Concurrency Model 屬性改為 `Fixed` 
        
        ![11concurrecnyfixed](https://user-images.githubusercontent.com/3851540/40586513-9104c96e-61f5-11e8-8ec6-6b08efc6bc9f.png)
4. 調整 View
    > 讓 `rowversion` 成為隱藏欄位

    ```cs
    @Html.HiddenFor(model=>model.rowversion)
    ```
1. 針對資料過時錯誤做特別處理
    
    ```cs
    try
    {
        await db.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException ex)
    {
        return Content($"<script>alert('資料已被他人修改，請重新編輯');document.location = '{Url.Action("Edit",user.Id)}'</script>");
    }
    ``` 
2. 實際效果
    
    ![12result](https://user-images.githubusercontent.com/3851540/40586514-91304c2e-61f5-11e8-88f7-355e254289c3.png)

## 心得
一開始在 SQL Server 上想加入 `rowversion` 一直找不到相關設定，查文件又說從 SQL Server 2008 開始就支援，讓我一度懷疑起是不是應該額外安裝什麼套件之類的，後來才發現原來與 `timestamp` 是同義詞，甚至還建議應該以 `rowversion` 為主，`timestamp` 會逐漸淘汰，但就連同屬自家工具的 SSMS (SQL Server Management Studio) 與 Visual Studio 都出現不同的呈現方式，就比較容易讓人混淆了

![13vsssms](https://user-images.githubusercontent.com/3851540/40586515-915d20be-61f5-11e8-96fe-e64b5de39d50.png)

# 參考資訊
1. [rowversion (Transact-SQL)](https://docs.microsoft.com/zh-tw/sql/t-sql/data-types/rowversion-transact-sql?view=sql-server-2017)
2. [EF Concurrency Mode Fixed + MVC](https://sellsbrothers.com/12699)