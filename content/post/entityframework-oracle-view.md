---
title: "Entity Framework 無法匯入 Oracle View？！"
date: 2017-12-13T02:58:00+08:00
lastmod: 2021-11-03T02:58:13+08:00
draft: false
tags: ["Entity Framework","Oracle"]
slug: "entityframework-oracle-view"
aliases:
    - /2017/12/entityframework-oracle-view.html
---
## Entity Framework 無法匯入 Oracle View？

同事反應在使用 Entity Framework EDMX 更新 Model 時一直無法將 View 加入(未看先猜是 primary key 所造成的問題，果不其然真的猜中XD)

DB View 的 priamry key 問題在 Entity Framework 使用上很常遇到，只是以往搭配的 DB 都是 SQL Server，想不到這次改成 Oracle，過去的做法就行不通了，就來看看 Oracle 該如何解決吧

## 錯誤訊息

1. edmx 選擇加入 View 但 designer 未出現

    ![1addview](https://user-images.githubusercontent.com/3851540/33902444-fa9d8b40-dfaf-11e7-99ff-549b580fdc70.png)

    ![2noview](https://user-images.githubusercontent.com/3851540/33902445-fac7d788-dfaf-11e7-9b8b-a8c7ef65ad70.png)

2. 編譯不會出現錯誤訊息

    ![3noerror](https://user-images.githubusercontent.com/3851540/33902446-faf29be4-dfaf-11e7-938a-6e52b12d3f16.png)

3. 使用 xml 編輯器打開 edmx 才會看到錯誤

    ```log
    Errors Found During Generation:
    warning 6013: The table/view 'TESTIDENTITY.ORGUNIT' does not have a primary key defined and no valid primary key could be inferred. This table/view has been excluded. To use the entity, you will need to review your schema, add the correct keys, and uncomment it.
    ```

    ![4error](https://user-images.githubusercontent.com/3851540/33902447-fb2082c0-dfaf-11e7-93a3-5c4c635b4342.png)

    * SQL Server 的解決方式可以參考保哥的文章 [解決 SQL Server 檢視表 (Views) 無法匯入 EDMX 的問題](https://blog.miniasp.com/post/2013/11/07/Entity-Framework-and-Primary-Keys-on-Views.aspx)，大意就是使用 SQL Server 的 `isnull` 語法調整 View，讓 Entity Framework 可以推斷出 primary key 即可
    * 我嘗試透過使用 Oracle 的 `nvl` 及 `COALESCE` 語法(效果等同 SQL Server 的 `isnull` 語法)都無法讓 Entity Framework 順利推斷出 primary key

## 只有特定 View 無法使用？

* 同個 db 中其他 View 可以正常匯入
* 無法匯入的類型：使用 `union all` 語法
  * 透過 table 的欄位(parentid) 來儲存父層資料

    ![10bu](https://user-images.githubusercontent.com/3851540/33902453-fc3e7c52-dfaf-11e7-964b-089ddd264e3c.png)

    ![11budata](https://user-images.githubusercontent.com/3851540/33902454-fc68716a-dfaf-11e7-9c9c-26ae6368ef67.png)

  * 使用 cte 遞迴取出資料

    ```sql
    WITH allunit ( id,parentid,Name,Level1)
    AS
    (select id,parentid,Name, 0 AS Level1
    from bu 
    where Parentid=0
    union all
    select a.id,a.parentid,a.Name, Level1 + 1
    from bu  a
    INNER JOIN allunit b
        ON b.id = a.parentid
    where  a.Parentid<>0 )
    SELECT *
    FROM allunit
        ```

    ![12viewdata](https://user-images.githubusercontent.com/3851540/33902456-fc987f04-dfaf-11e7-9c6e-cc63ce499cdf.png)

  * 原因尚待請教專業 DBA，可以先參考 [Is there a way to explicitely have a not-null column in a view](https://dba.stackexchange.com/questions/48169/is-there-a-way-to-explicitely-have-a-not-null-column-in-a-view)及[UNION of non-nullable columns is nullable](https://stackoverflow.com/questions/37551567/union-of-non-nullable-columns-is-nullable)

## 解決方式

**手動調整 edmx 內容**，以下紀錄調整方式

1. SSDL
    * 取消 EntityType 註解

        ```xml
        <EntityType Name="ORGUNIT">
            <Property Name="ID" Type="number" Precision="10" Scale="0" />
            <Property Name="PARENTID" Type="number" Precision="10" Scale="0" />
            <Property Name="NAME" Type="varchar2" MaxLength="20" />
            <Property Name="LEVEL1" Type="number" Precision="38" Scale="0" />
        </EntityType>
        ```

    * 加入 key

        ```xml
        <Key>
            <PropertyRef Name="ID" />
        </Key>
        ```

    * 將 key 加上 Nullable="false"

        ```cs
        Nullable="false"
        ```

    * 將 View 加至 EntityContainer 中，並指定 store type 為 Views

        ```xml
        <EntitySet Name="ORGUNIT" EntityType="Self.ORGUNIT" Schema="TESTIDENTITY" store:Type="Views" />
        ```

    * 完整內容

        ```xml
        <EntityType Name="ORGUNIT">
            <Key>
            <PropertyRef Name="ID" />
            </Key>
            <Property Name="ID" Type="number" Precision="10" Scale="0" Nullable="false" />
            <Property Name="PARENTID" Type="number" Precision="10" Scale="0" />
            <Property Name="NAME" Type="varchar2" MaxLength="20" />
            <Property Name="LEVEL1" Type="number" Precision="38" Scale="0" />
        </EntityType>
        <EntityContainer Name="ModelStoreContainer">
            <EntitySet Name="ORGUNIT" EntityType="Self.ORGUNIT" Schema="TESTIDENTITY" store:Type="Views" />
        </EntityContainer>
        ```

    ![5SSDL](https://user-images.githubusercontent.com/3851540/33902448-fb4a0816-dfaf-11e7-87f2-471059919e17.png)

2. CSDL

    > 這個可以在 model explorer 中加入，可以透過 xml 加入

    * 使用 xml
        * 將 View 加至 EntityContainer 中

            ```xml
            <EntitySet Name="OrgUnit" EntityType="Model.OrgUnit" />
            ```

        * 加入 EntityType 及屬性定義

            ```xml
            <EntityType Name="OrgUnit">
            <Key>
                <PropertyRef Name="ID" />
            </Key>
            <Property Name="ID" Type="Int32" Nullable="false" />
            <Property Name="NAME" Type="String" Nullable="false" />
            <Property Name="PARENTID" Type="Int32" Nullable="false" />
            <Property Name="LEVEL1" Type="Int32" Nullable="false" />
            </EntityType>
            ```

        ![6CSDL](https://user-images.githubusercontent.com/3851540/33902449-fb773bec-dfaf-11e7-9c31-2d10ffb3ac06.png)

    * 使用 model explorer
        * Add New Entity Type

            ![7addtype](https://user-images.githubusercontent.com/3851540/33902450-fba6c876-dfaf-11e7-8672-d9dd8c51dbf7.png)

        * Add Entity --> 指定程式用的名稱及 primary key

            ![8addeentity](https://user-images.githubusercontent.com/3851540/33902451-fbd8eab8-dfaf-11e7-8451-f065eb40580b.png)

        * Add New Property --> 新增其他屬性

            ![9addproperty](https://user-images.githubusercontent.com/3851540/33902452-fc0e1f3a-dfaf-11e7-8949-441aab05dc02.png)

3. C-S mapping content

    > 將 SSDL 與 CSDL mapping，一樣可以使用 xml 及 designer，建議使用 designer 較方便

    * Table Mapping

        ![13tablemapping](https://user-images.githubusercontent.com/3851540/33902852-5238bf36-dfb1-11e7-9420-e1e869d6e1a1.png)

    * Mapping detail

        > 名稱相同會自動對應，如果有對錯可以手動調整

        ![14mappingdetail](https://user-images.githubusercontent.com/3851540/33902853-52639292-dfb1-11e7-96ad-4ceb70069847.png)

## 心得

Entity Framework 搭配 Oracle，還是不像 SQL Server 那樣方便，需要踩的大大小小雷還不少，另外就是相關資料也不好找，常常找得我都會懷疑是真的不支援還只是沒有人分享做法 @@"

## 參考資訊

1. [解決 SQL Server 檢視表 (Views) 無法匯入 EDMX 的問題](https://blog.miniasp.com/post/2013/11/07/Entity-Framework-and-Primary-Keys-on-Views.aspx)
2. [Is there a way to explicitely have a not-null column in a view](https://dba.stackexchange.com/questions/48169/is-there-a-way-to-explicitely-have-a-not-null-column-in-a-view)
3. [UNION of non-nullable columns is nullable](https://stackoverflow.com/questions/37551567/union-of-non-nullable-columns-is-nullable)
