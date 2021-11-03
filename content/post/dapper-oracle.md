---
title: "Dapper 讀取 Oracle 資料"
date: 2017-07-04T22:58:00+08:00
lastmod: 2021-11-03T22:35:34+08:00
draft: false
tags: ["Dapper","Oracle"]
slug: "dapper-oracle"
aliases:
    - /2017/07/dapper-oracle.html
---
## Dapper 讀取 Oracle 資料

Dapper 身為輕量級 ORM 的神器，自從蔡煥麟老師 - [好用的微型 ORM：Dapper](http://www.huanlintalk.com/2014/03/a-micro-orm-dapper.html) 與 黑大 - [短小精悍的.NET ORM神器-- Dapper](http://blog.darkthread.net/post-2014-05-15-dapper.aspx) 撰文推廣後，讓愈來愈多人了解到 Dapper 的強大之處。而身為 ORM 愛用者，我也大量應用在實際專案上，使用上非常輕鬆寫意，直到後來工作上的 database 由 SQL Server 轉換為 Oracle

今天又遇到相同錯誤，紀錄一下，以後再遇到比較好抄XD

## 原始寫法

```cs
string cnstr = "Data source=YowkoOracle;User id=YowkoOracle_WRITE;Password=password;";
using (var cn = new System.Data.OracleClient.OracleConnection(cnstr))
{
    cn.Open();

    var result = cn.Query<string>(@"select TOKEN from TRIGGERTOKEN where ISACTIVE=1 and TOKEN=:token", new { token = "3D5998FASDF4531FDSAF64A4E" }).FirstOrDefault();
    result.Dump();
}
```

## OracleConnection is Obsolete

![1obsolete](https://user-images.githubusercontent.com/3851540/27827701-29a5bed8-60eb-11e7-97b5-4a5699087c25.png)

既然微軟已標記為過時，當然就不該繼續使用，避免出現 bug 跟可能的問題

## 改使用 `Oracle.DataAccess.Client`

* 使用 x86 版本

  * NuGet 下載

    ![2x86](https://user-images.githubusercontent.com/3851540/27827703-29a8e1bc-60eb-11e7-9bdc-272133f78fc2.png)

  * ID：Oracle.DataAccess.x86

* ~~實際 x64 版本無法成功連線~~ AnyCPU 版
  * NuGet 下載

    ![3x64](https://user-images.githubusercontent.com/3851540/27827700-299f4a08-60eb-11e7-8004-2ff2c0963c96.png)

  * ID：Oracle.ManagedDataAccess
  * namespace 與 x86 不同，要留意

    ```cs
    using (var cn = new Oracle.ManagedDataAccess.Client.OracleConnection(cnstr))
    ```

  * 錯誤訊息：`ORA-12154: TNS: 無法解析指定的連線 ID`

    ![4error](https://user-images.githubusercontent.com/3851540/27827702-29a71b20-60eb-11e7-92b1-01373d547b8b.png)

    * 當時未能查明原因，推估應是測試當下 tns.ora 設定有誤造成
  * </del><span style="color:red"> 2017/11/18 重新 review 後，發現內文有錯，正確用法連結：</span> [Dapper 讀取 Oracle 資料 - 更新版 (使用 Oracle.ManagedDataAccess )](/dapper-oracle-manageddataaccess)

  * 完整程式碼

    ```cs
    string cnstr = "Data source=YowkoOracle;User id=YowkoOracle_WRITE;Password=password;";
    using (var cn = new Oracle.DataAccess.Client.OracleConnection(cnstr))
    {
        cn.Open();
        var result = cn.Query<string>(@"select TOKEN from TRIGGERTOKEN where ISACTIVE=1 and TOKEN=:token", new { token = "3D5998FASDF4531FDSAF64A4E" }).FirstOrDefault();
            result.Dump();
    }
    ```

## 參數與 SQL Server 用法不同

Oracle 與 SQL Server 在 sql secript 中使用參數寫法不同

1. Oracle 寫法為 `:Variable`

    ```cs
    var result = cn.Query<string>(@"select TOKEN from TRIGGERTOKEN where ISACTIVE=1 and TOKEN=:token", new { token = "3D5998FASDF4531FDSAF64A4E" }).FirstOrDefault();
    ```

2. SQL Server 寫法為 `@Variable`

    ```cs
    var list = cn.Query<Guid>("SELECT TOKEN FROM [TRIGGERTOKEN] where isActive=1 and token=@token", new { token = "3D5998FASDF4531FDSAF64A4E" }).FirstOrDefault();
    ```

## 心得

不知道是不是我不熟悉 Oracle 的關係，覺得 dapper 連線 Oracle 要注意的東西還不少，這個不做筆記，我想下次我還是會花不少時間

## 參考資訊

1. [Dapper連接Oracle](http://www.cnblogs.com/ushou/p/3359973.html)
2. [好用的微型 ORM：Dapper](http://www.huanlintalk.com/2014/03/a-micro-orm-dapper.html)
3. [短小精悍的.NET ORM神器-- Dapper](http://blog.darkthread.net/post-2014-05-15-dapper.aspx)
4. [Dapper 讀取 Oracle 資料 - 更新版 (使用 Oracle.ManagedDataAccess )](/dapper-oracle-manageddataaccess)
