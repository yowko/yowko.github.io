---
title: "取得 Entity Framework 存取 DB 時的實際 SQL Script"
date: 2017-10-29T15:48:00+08:00
lastmod: 2021-11-03T15:48:17+08:00
draft: false
tags: ["Entity Framework"]
slug: "entityframework-log-sql"
aliases:
    - /2017/10/entityframework-log-sql.html
---
## 取得 Entity Framework 存取 DB 時的實際 SQL Script

使用 ORM 的好處之一就是可以專注在 object 控制，不用擔心 sql 語法不熟悉而寫不出正確的 sql script，也可以省下與不同 sql 語法間 (t-sql、pl/sql) 奮鬥的時間跟精力。

不過 ORM 終究不是萬靈丹，想要針對 sql script 有更精確的控制或是調校時(像是希望吃到特定 index 或是想要使用特定 hint) 還是需要自行處理，而在自行撰寫 sql 之前，當然要先看看 Entity Framework 產生出 script 與實際目標差多少，再決定該如何進行下一步，所以馬上來看看該如何取得實際 sql 吧

<span style="color:red">Entity Framework 不同版本有不同做法，以下使用 Entity Framework 6.X 版本</span>

## 做法一：使用 Context Log 屬性

> 以下做法會將所有與 DB 的操作都紀錄下來

* console project 適用

    ```cs
    using (var db = new Entities())
    {
        db.Database.Log = Console.Write;
        
        // 實際執行動作 ...
    }
    ```

    ![1console](https://user-images.githubusercontent.com/3851540/32141591-4d0a0d22-bcbf-11e7-8bcf-fd4d0a3b222f.png)

* web project 適用

    ```cs
    using (var db = new Entities())
    {
        db.Database.Log = (log) => db.Database.Log = (log) => System.Diagnostics.Debug.WriteLine(log);
        
        // 實際執行動作 ...
    }
    ```

  * 因為 web project 沒有 console 可以輸出，但可以透過 debug 時的 output 視窗來做輸出

    ![2web](https://user-images.githubusercontent.com/3851540/32141592-4d3835b2-bcbf-11e7-8c95-471e651c2994.png)

  * 也適用於 console project

    ![3console2](https://user-images.githubusercontent.com/3851540/32141593-4d61feba-bcbf-11e7-82a5-0cefb8c5ee9b.png)

* 優點：可以精準控制 (稍後會介紹)
* 缺點：需要調整程式

## 做法二：直接 output LinQ 語法

* 使用方式

    ```cs
    using (var db = new Entities())
    {
        Console.WriteLine(db.USERPROFILE.ToString());
    }
    ```

    ![4tostring](https://user-images.githubusercontent.com/3851540/32141594-4d895de8-bcbf-11e7-919d-d3ef757b7973.png)

* 優點：可精準控制需要 log 的內容

* 缺點：限制不少，像是 `FirstOrDefault()` 執行完已經是特定 object 就會將 object type ToString，但其他還是 IQuerable 的語法則是可以正常使用

## 做法三：在 config 檔案中加入 interceptors 設定

<span style="color:red">僅適用 Entity Framework 6.1 以上版本</span>

* 在 `config` 的 `entityFramework` 區段加上 `interceptors` 設定

    ```xml
    <interceptors>
        <interceptor type="System.Data.Entity.Infrastructure.Interception.DatabaseLogger, EntityFramework">
            <parameters>
                <parameter value="{儲存檔案位置}" />
            </parameters>
        </interceptor>
    </interceptors>
    ```

* 實際範例

    ```xml
    <entityFramework>
        <defaultConnectionFactory type="System.Data.Entity.Infrastructure.SqlConnectionFactory, EntityFramework" />
        <providers>
            <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
        </providers>
        <interceptors>
            <interceptor type="System.Data.Entity.Infrastructure.Interception.DatabaseLogger, EntityFramework">
                <parameters>
                <parameter value="C:\temp\DataAccessLogOutput.txt" />
                </parameters>
            </interceptor>
        </interceptors>
    </entityFramework>
    ```

    ![5interceptor](https://user-images.githubusercontent.com/3851540/32141595-4db3d4b0-bcbf-11e7-9bee-98eea794ebba.png)

* 優點：不需要改程式
* 缺點：無差別 log 所有 sql

## 精準紀錄特定 db 操作

程式與 db 溝通相當頻繁，無差別全部 log 很容易造成雜訊過多，讓 debug 變得沒效率，此時可以透過這個方式只紀錄需要的部份

* 僅在需要紀錄的位置前加上 `db.Database.Log = Console.Write;`，執行動作後就移除

    ```cs
    using (var db = new Entities())
    {
        //其他操作
        db.Database.Log = Console.Write;
        //想要紀錄的操作
        db.Database.Log = null;
        
        //其他操作
    }
    ```

* 實際範例

    ```cs
    using (var db = new Entities())
    {
        Console.WriteLine(db.USERPROFILE.FirstOrDefault().ADDR);
        
        db.Database.Log = Console.Write;
        Console.WriteLine(db.USERPROFILE.FirstOrDefault().NAME);
        db.Database.Log = null;
        
        Console.WriteLine(db.USERPROFILE.FirstOrDefault().BIRTHDAY);
        Console.ReadLine();
    }
    ```

    ![6lspecificog](https://user-images.githubusercontent.com/3851540/32141596-4ddf050e-bcbf-11e7-8ba1-f887e095d493.png)

* 優點：精準控制需要 log 的內容
* 缺點：需要手動處理

## 搭配既有 log 機制

專案都有自己的 log 機制，希望在 production 環境也將 sql log 下來供 debug 用，最好的做法就是與既有專案 log 機制配合，不要另外處理以免造成 log 資訊散落各處，徒增 debug 難度

* 將既有 logger 指定給 Context Log 屬性即可

    ```cs
    // logger 為既有 log 機制
    db.Database.Log = (log)=> logger.Debug(log);
    // 想要 log 的操作
    ```

* 實際範例

    > 以下使用 nlog 搭配寫入 file 的方式來示範

    ```cs
    static  ILogger logger =LogManager.GetCurrentClassLogger();
    static void Main(string[] args)
    {
        using (var db = new Entities())
        {
            Console.WriteLine(db.USERPROFILE.FirstOrDefault().ADDR);
            
            db.Database.Log = (log)=> logger.Debug(log);
            Console.WriteLine(db.USERPROFILE.FirstOrDefault().NAME);
            db.Database.Log = null;
            
            Console.WriteLine(db.USERPROFILE.FirstOrDefault().BIRTHDAY);
        }
    }
    ```

    ![7nlog](https://user-images.githubusercontent.com/3851540/32141597-4e0a2f7c-bcbf-11e7-8975-2d038b740779.png)

* 優點：可以精準控制
* 缺點：需要調整程式

## 心得

原本只是因為有陣子沒用到 Entity Framework 紀錄產出的 sql 功能，打算加深印象隨手紀錄一下，想不到查資料過程中愈記愈多XD，比預期中多花了好幾個小時來測試以及確認，讓今天的計劃大 delay，不過至少對 Entity Framework log 的部份有更深印象，也是非常有收獲

## 參考資訊

1. [Entity Framework Logging and Intercepting Database Operations (EF6 Onwards)](https://msdn.microsoft.com/en-us/library/dn469464%28v=vs.113529.aspx)
2. [How do I view the SQL generated by the Entity Framework?](https://stackoverflow.com/questions/1412863/how-do-i-view-the-sql-generated-by-the-entity-framework)
