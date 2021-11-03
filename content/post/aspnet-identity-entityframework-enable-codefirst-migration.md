---
title: "ASP.NET Identity 搭配 Entity Framework 啟用 Code First Migration"
date: 2017-02-04T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET Identity","Entity Framework"]
slug: "aspnet-identity-entityframework-enable-codefirst-migration"
aliases:
    - /2017/02/aspnet-identity-entityframework-enable-codefirst-migration.html
---
## ASP.NET Identity 搭配 Entity Framework 啟用 Code First Migration

ASP.NET Identity 是個方便的 member system, 原生就是搭配 Entity Framework 的 code first 機制，如果需要針對 ASP.NET Identity 做一些擴充，就需要啟用 Code First Migration，過去曾經做過幾次印象不深，近期專案用到趁著這個機會好好紀錄一下

## 錯誤訊息

1. 訊息內容

    ```log
    The model backing the 'ApplicationDbContext' context has changed since the database was created. Consider using Code First Migrations to update the database (http://go.microsoft.com/fwlink/?LinkId=238269).
    ```

2. 錯誤截圖

    ![1error](https://cloud.githubusercontent.com/assets/3851540/22261474/1c1cfe64-e2a8-11e6-9ea8-59d950c8041a.png)

## 啟用 Code First Migration

1. Tools --> NuGet Package Manager --> Package Manager Console

    ![2pmc](https://cloud.githubusercontent.com/assets/3851540/22261475/1c527576-e2a8-11e6-832c-2bd0b11f33f1.png)

2. 執行指令 `Enable-Migrations`

    ![3execute](https://cloud.githubusercontent.com/assets/3851540/22261477/1c840dfc-e2a8-11e6-93bd-50bfbbd5fcd7.png)

3. `Migrations` 資料夾增加兩個檔案

    ![4MIGRATION](https://cloud.githubusercontent.com/assets/3851540/22261476/1c80c3f4-e2a8-11e6-9413-678768299a1f.png)

    - 3-1. Configuration.cs
        - 設定是否啟用自動整合
        - 需要於 DB 建立時，自動加入特定資料，可以寫在 Seed 中
    - 3-2. {TimeStamp}_InitialCreate.cs
        - DB 已存在才會出現這個檔案
        - 用來描述 DB 已存在的物件

## Migration command

1. Add-Migration
    - Package Manager Console 執行 `Add-Migration {MigrationName}`
    - 比對與上次變更的差異，建立變更描述
    - `Migrations` 資料夾會建立 `{TimeStamp}_{MigrationName}.cs`
    - 有 Up (描述與之前版本的異動內容)與 Down (描述如何從 Up 執行後回到目前狀態)

2. Update-Database
    - Package Manager Console 執行 `Update-Database`
    - 將未更新的變更皆套用至資料庫
    - 加上 `-Verbose` 可以看到 migration 實際執行的 SQL
    - 可以在 `{TimeStamp}_{MigrationName}.cs` 的 Up 中客製行為(e.g. 加入欄位定義或是指定 primary-key ....)

## Migrate 到特定版本

1. 可用來降版
    - `Update-Database –TargetMigration:{MigrationName}`
    - 會執行與目標版本間的所有 Down 指令

2. 可用來回覆至空白資料庫
    - Update-Database -TargetMigration:$InitialDatabase

## 取得可以執行的 SQL

 > 用來取得無法直接透過 Visual Studio 執行變更環境的 SQL

1. 取得所有未套用的變更

    ```cmd
    Update-Database -Script
    ```

2. 取得特定版本

    ```cmd
    Update-Database -Script -SourceMigration: {MigrationName1} -TargetMigration: {MigrationName2}
    ```

3. 取得從空白資料庫到指定版本間的變更

    ```cmd
    Update-Database -Script -SourceMigration: $InitialDatabase -TargetMigration: {MigrationName}
    ```

## Auto Migration

> 執行程式時，如果偵測到資料庫變更，自動套用

1. using System.Data.Entity;

    > `Database` 與 `MigrateDatabaseToLatestVersion` 的 namespace

2. using {projectName}.Migrations;

    > `Configuration` 的 namespace

3. using {projectName}.Models;

    > {ContextName} 的 namespace

4. Database.SetInitializer(new MigrateDatabaseToLatestVersion<{ContextName}, Configuration>());

    > MVC 可以加在 Global.asax 的 Application_Start()

## 參考資料

1. [Data Access and Storage > 學習園地 > Entity Framework > 開始使用 > Code First 移轉](https://msdn.microsoft.com/zh-tw/data/jj591621)
