---
title: "Dapper 讀取 Oracle 資料 - 更新版 (使用 Oracle.ManagedDataAccess)"
date: 2018-01-14T17:20:00+08:00
lastmod: 2018-10-02T17:20:06+08:00
draft: false
tags: ["套件","Dapper","Oracle"]
slug: "dapper-oracle-manageddataaccess"
aliases:
    - /2018/01/dapper-oracle-manageddataaccess.html
---
# Dapper 讀取 Oracle 資料 - 更新版 (使用 Oracle.ManagedDataAccess)
之前筆記 [Dapper 讀取 Oracle 資料](https://blog.yowko.com/2017/07/dapper-oracle.html) 提到使用 ~~x64 版本的 oracle client - Oracle.ManagedDataAccess 無法成功連線~~，<span style="color:red">這句話是錯誤的</span>，一來 `Oracle.ManagedDataAccess` 本身是 32位元，另外就是 `Oracle.ManagedDataAccess` 絕對可以正常連線 oracle 讀寫資料

就來看看如何使用 `Oracle.ManagedDataAccess` 搭配 `dapper` 連線 oracle 吧

## 安裝 `Oracle.ManagedDataAccess`

>使用 NuGet 搜尋並安裝

![1oraclemanaged](https://user-images.githubusercontent.com/3851540/34914473-5a157012-f94e-11e7-9eab-d88db9787ab4.png)

## 安裝 `dapper`

>使用 NuGet 搜尋並安裝

![2dapper](https://user-images.githubusercontent.com/3851540/34914474-5a3f4f86-f94e-11e7-9e10-ee191efb2560.png)

## 程式碼

```CS
//oracle 連線字串
string cnstr = "Data source=localhost;User id=TestIdentity;Password=password;";
//dapper 建立連線物件
using (var cn = new Oracle.ManagedDataAccess.Client.OracleConnection(cnstr))
{
    //開啟連線
    cn.Open();
    //執行查詢
    var result = cn.Query<string>(@"select name from menu where isdelete=:isdelete", new {isdelete=0});
    //將結果印出
    result.Dump();
}
```

## 心得

與之前使用 `Oracle.DataAccess.Client` 最大的差別只在建立連線時使用的 namespace 的不同，當然背後隱藏了很多細節：(以下參考黑大說明 - [Managed ODP.NET簡介](http://blog.darkthread.net/post-2015-03-31-managed-odp-net.aspx))

1.  可以直接使用 Oracle.ManagedDataAccess ，無需安裝 Oracle Client
2.  Oracle.ManagedDataAccess 不必區分 32位元及 64位元


現在回想之前筆記時，已經忘記為什麼當時內容錯誤原因，當然不懂是主因但畢竟在真正紀錄時我都會再做過測試，確認結果才做筆記，當時內容有問題也代表測試也出錯，但現在完全沒有印象當時出了什麼差錯，不過也多虧這次經驗讓我更覺得該好好做筆記，至少有印象曾經寫錯還能好好檢視當時錯誤原因並改進，不是一直帶著錯誤的觀念持續錯下去

# 參考資訊

1.  [Dapper 讀取 Oracle 資料](https://blog.yowko.com/2017/07/dapper-oracle.html)
2.  [Managed ODP.NET簡介](http://blog.darkthread.net/post-2015-03-31-managed-odp-net.aspx)
