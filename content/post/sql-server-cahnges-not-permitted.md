---
title: "SQL Server 無法修改資料表定義？！"
date: 2017-06-23T21:30:00+08:00
lastmod: 2018-09-23T21:30:12+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-cahnges-not-permitted"
aliases:
    - /2017/06/sql-server-cahnges-not-permitted.html
---
# SQL Server 無法修改資料表定義？！
沒辦法直接修改 SQL Server 資料表定義的問題，說常見也不是很常見，但也不算罕見，畢竟通常只需要設定一次，剛好我常常安裝新的測試用 SQL Server 所以遇過好幾次，這次隔比較久，就花了比較多時間才找到設定，特別筆記一下

## 錯誤訊息
- 訊息內容
    
    >Saving changes is not permitted.The changes you have made reuire the following tables to be dropped and re-created.You have either made changes to a table that can't be re-created or enabled the option Prevent saving changes that require the table to be re-created.
    
- 錯誤截圖
    
    ![1errormsg](https://user-images.githubusercontent.com/3851540/27478402-188e72ea-5842-11e7-81f4-00621a85739f.png)

* 不允許儲存變更，因為您所做的變更需要卸除及重新建立所列出的資料表

## 可能需要重新建立資料表的操作

1.  將新 column 插入 table 的中間
2.  移除 column
3.  改動 column 的 Null 屬性
4.  改變 column 的順序
5.  改變 column 的資料類型


## 發生原因

異動資料表的中繼資料結構時，因為 table 可能需要重建。而重建 table 時，可能會造成中繼資料的異常和資料遺失，所以預設不允許會造成 table 重建的 table 異動

## 如何關閉

1.  SSMS 主選單上的 Tools --> Options...

    ![2tools](https://user-images.githubusercontent.com/3851540/27478403-188f8ad6-5842-11e7-8481-4a3f35cc81a6.png)

2.  Designers --> 取消勾選 Prevent saving changes that require the table to be re-created

    ![3uncheck](https://user-images.githubusercontent.com/3851540/27478404-189175e4-5842-11e7-8ce6-c9c83f9edcfa.png)

## 建議作法

直接從 table desginer 異動 table design 有造成資料遺失的風險，官方建議採用 SQL statement 來異動 table

*   範例：將 table column 改為允許 NULL

    > `alter table MyTable alter column CreateDate datetime NULL`

## 心得

我自己通常是在開發階段需要反覆地修改 table design，所以會刻意關關以便於開發階段的調整，加上開發環境的資料重要性較低也較少，如果出現問題影響有限，至於其他環境 是否要關閉請自行衡量實際情況(雖然我沒有真的發生資料遺失的情形，但官方都說有風險，請不要鐵齒~~)

# 參考資訊

1.  [Save (Not Permitted) Dialog Box](https://docs.microsoft.com/en-us/sql/ssms/visual-db-tools/save-not-permitted-dialog-box?WT.mc_id=DOP-MVP-5002594)
2.  [當您嘗試在 SQL Server 中儲存的資料表時，出現錯誤訊息: 「 儲存變更並不允許"](https://support.microsoft.com/zh-hk/help/956176/error-message-when-you-try-to-save-a-table-in-sql-server-saving-changes-is-not-permitted)
