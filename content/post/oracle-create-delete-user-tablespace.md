---
title: "Oracle 快速刪除並重建使用者"
date: 2017-10-24T23:59:00+08:00
lastmod: 2018-09-27T23:59:15+08:00
draft: false
tags: ["Oracle"]
slug: "oracle-create-delete-user-tablespace"
aliases:
    - /2017/10/oracle-create-delete-user-tablespace.html
---
# Oracle 快速刪除並重建使用者
之前工作大多是使用 MS SQL Server，現在任職公司則是使用 Oracle 為主力 DB。前陣子工作內容都是放在現有專案的改善及維護，平常跟 DB 接觸頂多只是查詢資料，而最近恰逢要建置新專案，才讓我有比較多機會可以摸到 Oracle。

專案建置過程中隨著功能細節逐漸成型讓 user 有不同想法並調整需求加上規劃初期時未考慮到的問題，常常會需要頻繁調整 db table 架構或是欄位，之前使用 sql server 時一般都是透過 localdb 來隔離調整，避免異動 db 時影響其他團隊成員開發，但 oracle 並沒有 localdb 這樣的機制，為此我就在 local 開發機上安裝了 oracle xe 版本來因應。

為了讓系統上線時可以一切順利，便嘗試將本機環境也設定與正式環境一致(主要是 tablesapce 與 user 相關設定)，就在這個嘗試建立、設定、重新調整的過程中，順便紀錄一下使用方式

## 原本方式

```sql
-- 移除 TEST 使用者的所有物件
drop user TEST cascade ;

-- 建立 TEST 使用者並指定密碼為 `password` 、預設儲存的 tablespace 為 `USERS`
create user TEST identified by password default tablespace USERS;
-- 給予 TEST 使用者 DBA 權限
grant DBA to TEST;

-- 移除名為 "TEST_TBS" 的 tablespace 並刪除其內容及檔案;
drop tablespace "TEST_TBS" including contents and datafiles CASCADE CONSTRAINTS;
-- 建立名為 "TEST_TBS" 的 tablespace 並指定檔案位置為 `C:\oraclexe\oracledata\data.dbf`及檔案大小為 50mb
CREATE TABLESPACE "TEST_TBS" DATAFILE 'C:\oraclexe\oracledata\data.dbf' SIZE 50M;
```

## 建議作法

上面的寫法被 dba 同事看到，立馬引起 dba 的高度驚恐，主要是我不曉得要給什麼權限，東卡西卡的搞好久，乾脆給 dba 權限最快，但同事立馬給了我正確的權限名稱，個人秉持著 `最小知識原則` 一定要使用正確的方式嘛

```sql
-- 移除 TEST 使用者的所有物件
drop user TEST cascade ;

-- 建立 TEST 使用者並指定密碼為 `password` 、預設儲存的 tablespace 為 `USERS`
create user TEST identified by password default tablespace USERS;
-- 給予 TEST 使用者 `Resource`、`connect` 權限
grant Resource,connect to TEST;

-- 移除名為 "TEST_TBS" 的 tablespace 並刪除其內容及檔案;
drop tablespace "TEST_TBS" including contents and datafiles CASCADE CONSTRAINTS;
-- 建立名為 "TEST_TBS" 的 tablespace 並指定檔案位置為 `C:\oraclexe\oracledata\data.dbf`及檔案大小為 50mb
CREATE TABLESPACE "TEST_TBS" DATAFILE 'C:\oraclexe\oracledata\data.dbf' SIZE 50M;
```

## 心得

原來只要給予 `Resource`、`connect` 原則上就可以應付大部份使用情境了，這果然還是得要熟悉 orcle 才知道的呀

幸虧有同事提點讓我學到正確的方式，其實本機使用也不是大問題，但就是技術人的小小潔僻，如果只是擔心其他人看到覺得外行也就算了，但我就是過不了自己那關，看著 sql 都覺得不應該 grant dba 權限，內心糾結了好久終於在同事的幫助下搞定了

# 參考資訊

1.  [帳號, 密碼, 權限與角色(轉)](http://oracled2k.pixnet.net/blog/post/24680289--%E5%B8%B3%E8%99%9F%2C-%E5%AF%86%E7%A2%BC%2C-%E6%AC%8A%E9%99%90%E8%88%87%E8%A7%92%E8%89%B2%28%E8%BD%89%29)
2.  [Oracle創建及刪除使用者](http://fecbob.pixnet.net/blog/post/43275970-oracle%E5%89%B5%E5%BB%BA%E5%8F%8A%E5%88%AA%E9%99%A4%E4%BD%BF%E7%94%A8%E8%80%85)
3.  [oracle drop表空間中的datafile](http://fecbob.pixnet.net/blog/post/43277980-oracle-drop%E8%A1%A8%E7%A9%BA%E9%96%93%E4%B8%AD%E7%9A%84datafile)
4.  [TableSpace觀念與建立、更改、刪除](http://yehyenping.blogspot.tw/2013/06/tablespace.html)
