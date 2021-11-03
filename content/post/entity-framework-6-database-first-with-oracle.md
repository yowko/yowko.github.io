---
title: "使用 Entity Framework 連線 Oracle - Database First"
date: 2017-10-01T00:47:00+08:00
lastmod: 2021-11-03T00:47:11+08:00
draft: false
tags: ["Entity Framework","Oracle"]
slug: "entity-framework-6-database-first-with-oracle"
aliases:
    - /2017/10/entity-framework-6-database-first-with-oracle.html
---
## 使用 Entity Framework 連線 Oracle - Database First

之前工作一直使用 Entity Framework 加速專案開發速度，主要是因為過去公司都使用 MS-SQL，與 Entity Framework 整合度很高，配合 Visual Studio Scaffolding 功能，可以很迅速地將專案骨架展開，讓工程師可以直接進入商業邏輯及其他需求功能開發，是非常方便好用的工具，但 Entity Framework 有不少缺點：像是無法控制轉譯後的 SQL 及預設的 change tracking 讓資料使用情境的效能大降都使得不少人卻步

最近為了加速專案開發，打算透過 Entity Framework 搭配 Oracle 先長出專案的主框架，過程中遇到不少問題，順手紀錄一下

## 環境說明

1. Visual Studio 2017
2. Entity Framework 6
3. .Net Framework 4.6.2
4. Oacle 11g XE
5. Windows 10

## 安裝 Oracle Developer Tools for Visual Studio

請至 Oracle 官網下載對應的版本 [Oracle Data Access Components (ODAC) for Windows Downloads](http://www.oracle.com/technetwork/topics/dotnet/downloads/index.html)，下列示範將使用 [Oracle Developer Tools for Visual Studio 2017 - MSI Installer](http://www.oracle.com/technetwork/topics/dotnet/downloads/odacmsidownloadvs2017-3806459.html)，這會加入 Oracle Data provider，讓 Entity Framework 能與 Oracle 溝通

* 安裝前

    ![1nooracle](https://user-images.githubusercontent.com/3851540/31047626-caa18a5c-a640-11e7-86ce-1574c091b919.png)

* 安裝後

    ![2oracle](https://user-images.githubusercontent.com/3851540/31047627-caa2ba4e-a640-11e7-8e01-89b86fceb16f.png)

* 如果這邊看不出用途沒關係，後面會有更詳細的說明，記得先安裝就對了

## 安裝 Oracle Driver

安裝 `Oracle.ManagedDataAccess.EntityFramework` 支援 Entity Framework 6，順帶安裝的 `Oracle.ManagedDataAccess` 則是 oracle 官方的 data provider

![3oracleMDAEF](https://user-images.githubusercontent.com/3851540/31047628-caa610a4-a640-11e7-94d2-dab110f47f6c.png)

![4oracleMDAEF](https://user-images.githubusercontent.com/3851540/31047629-caaa4f20-a640-11e7-98d7-e57d73aaf3c9.png)

## 加入 ADO.NET Entity Data Model (.edmx)

1. 在 `Model` 加入新項目

    > 專案 `Model` 資料夾上按右鍵 --> Add --> New Item...

    ![5addedmx](https://user-images.githubusercontent.com/3851540/31047631-cac94eca-a640-11e7-8763-13da5dc51f7e.png)

2. 選擇 `ADO.NET Entity Data Model`

    > Data --> ADO.NET Entity Data Model

    ![6addedmx2](https://user-images.githubusercontent.com/3851540/31047630-cac859fc-a640-11e7-9d04-f82a158bdbec.png)

3. 選擇 Model 內容

    ![7modelcontent](https://user-images.githubusercontent.com/3851540/31047632-caca9078-a640-11e7-900b-b2215e2f595a.png)

4. 設定連線資料

    ![8connection1](https://user-images.githubusercontent.com/3851540/31047633-cacbca06-a640-11e7-95fd-57de75245da8.png)

    ![9connection2](https://user-images.githubusercontent.com/3851540/31047634-cad2eed0-a640-11e7-9f61-233a2e5d8c42.png)

    ![10connection3](https://user-images.githubusercontent.com/3851540/31047635-cad6474c-a640-11e7-9238-6f3891135898.png)

    ![11connection4](https://user-images.githubusercontent.com/3851540/31047636-caf204b4-a640-11e7-9c7f-e2e48e91d345.png)

    ![12connection5](https://user-images.githubusercontent.com/3851540/31047637-caf29366-a640-11e7-920d-e03f03dff1f2.png)

5. 選擇要轉換為 model 為資料庫物件

    ![13connection6](https://user-images.githubusercontent.com/3851540/31047618-ca75eeb0-a640-11e7-8170-428f369169a5.png)

6. 儲存 `.edmx` 並重新編譯

    ![14edmx](https://user-images.githubusercontent.com/3851540/31047620-ca77c726-a640-11e7-97bc-0db33e847af0.png)

## 使用 Scaffolding 產生程式碼

1. 加入 Controller

    > `Controllers` 資料夾上按右鍵 --> Add --> Controller...

    ![15controller1](https://user-images.githubusercontent.com/3851540/31047619-ca76792a-a640-11e7-93e9-4ba4510d5b5f.png)

2. 選擇 Scaffold template

    ![16controller2](https://user-images.githubusercontent.com/3851540/31047621-ca792be8-a640-11e7-9767-0d20a384a7e3.png)

3. 選擇 model、context 並指定名稱

    ![17controller3](https://user-images.githubusercontent.com/3851540/31047622-ca7b17be-a640-11e7-82ed-62505b295262.png)

## 可能問題

1. 無法使用 Entity Framework 6

    ```log
    An Entity Framework database provider compatible with the laest version of Entity 
    Framework could not be found for your data connection. If you have already installed a 
    compatible provider, ensure you have rebuilt your project performing this action. 
    Otherwise, exit this wizard, install acompatible provider and rebuild your project before 
    performing this action.
    ```

    ![18noef6](https://user-images.githubusercontent.com/3851540/31047623-ca7f4af0-a640-11e7-86e8-8036265b7f02.png)

    * 預設安裝的 data provider 僅支援 Entity Framework 5.0
    * 解決方式：安裝 `Oracle.ManagedDataAccess.Entity Framework`

2. 無法使用 Scaffolding

    ```log
    Error
    
    There was an error running the selected code gernerator:
    'Could not load file or assmbly 'Entity Framework,
    Version=5.0.0.0,Culture=neutual,
    PublicKeyToken=b77a5c561934e089' or one of its dependencies.
    The located assembly's manifest definition does not match the
    assembly reference.(Exception from HRESULT:0x80131040)'
    ```

    ![19noscaffold](https://user-images.githubusercontent.com/3851540/31047624-ca9fd2d4-a640-11e7-9351-765bdae231eb.png)

    * 重新檢查專案設定，會發現 Entity Framework 被升級為 6.x 版本，建立時說不能用 Entity Framework 6，現在又幫我升級XD
    * 解決方式：安裝 `Oracle.ManagedDataAccess.EntityFramework`

3. 無法使用 Scaffolding

    ```log
    Error
    
    There was an error running the selected code generator:
    'Unable to retrieve metadata for 
    'xxxx'.Schema specified is not valid. Errors:.........
    ```

    ![20typeerror](https://user-images.githubusercontent.com/3851540/31047625-ca9fdfa4-a640-11e7-8eca-0ad9a3feeda6.png)

    * `.edmx` 中定義的型別不正確
    * 解決方式：重新產生 `.edmx`

## 心得

不知道是不是我被微軟養大胃口了，覺得與 Oracle 的整合好麻煩，一下缺東一下缺西的，安裝說明跟錯誤訊息都不是很明確，不是很好 debug，使用體驗很差，明明設定步驟不多，操作也不複雜，但著實花了我一整天才搞定，希望這關過了後面可以順利些

## 參考資訊

1. [Oracle Data Access Components (ODAC) for Windows Downloads](http://www.oracle.com/technetwork/topics/dotnet/downloads/index.html)
2. [Entity Framework 6, database-first with Oracle](https://csharp.today/entity-framework-6-database-first-with-oracle/)
