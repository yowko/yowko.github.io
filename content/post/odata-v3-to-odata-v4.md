---
title: "如何將 OData v3 升級為 OData v4"
date: 2017-10-15T12:24:00+08:00
lastmod: 2021-08-18T12:24:23+08:00
draft: false
tags: ["ASP.NET Web API","OData"]
slug: "odata-v3-to-odata-v4"
aliases:
    - /2017/10/odata-v3-to-odata-v4.html
---
## 如何將 OData v3 升級為 OData v4

ASP.NET Web API 支援最高版本 OData 是 v4，但只能透過手動的方式建立，OData v3 就支援利用 scaffolding 來長出各個 table 的 OData endpoint ，加上 OData v3 到 OData v4 就連 namespace 都變了，NuGet package 也不同，無法單純升級 NuGet package 就可以搞定的，雖然可以手動慢慢建立也是解法，但身為一個高效工程師，一個個手動新增這種苦工絕對不是我們的作風，就來看看可以怎麼快速地將 OData v3 升級為 OData v4

## 前提設定

已經使用 EntityFramework 設定與 DB 連線，並透過預設 scaffolding template 建立 ASP.NET Web API OData v3 controller

## 升級作法

1. 安裝 `Microsoft.AspNet.OData` 套件

    > 這是專屬 OData v4 的套件名稱

2. 修改 controller
    * 使用 odata v4 namespace
        * 原語法

            ```cs
            using System.Web.Http.OData;
            ```

        * 新語法

            ```cs
            using System.Web.OData;
            ```

    * 調整為 odata v4 寫法
        * 原語法

            > 用於 `Put` 與 `Patch`

            ```cs
            Validate(patch.GetEntity());
            ```

        * 新語法

            ```cs
            Validate(patch);
            ```

3. 修改 `WebApiConfig` OData 註冊
    * 使用 odata v4 namespace
        * 原語法

            ```cs
            using System.Web.Http.OData.Builder;
            using System.Web.Http.OData.Extensions;
            ```

        * 新語法

            ```cs
            using System.Web.OData.Builder;
            using System.Web.OData.Extensions;
            ```

    * 調整為 odata v4 註冊語法
        * 原語法

            ```cs
            config.Routes.MapODataServiceRoute("odata", "odata", builder.GetEdmModel());
            ```

        * 新語法

            ```cs
            config.MapODataServiceRoute("odata", "odata", builder.GetEdmModel());
            ```

4. 可能出現：  `The entity 'xxx' does not have a key defined.`
    * 錯誤訊息

        ```txt
        The entity 'Orders' does not have a key defined.
        
        Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 
        
        Exception Details: System.InvalidOperationException: The entity 'Orders' does not have a key defined.
        
        Source Error: 
            
        Line 47: 
        Line 48:             
        Line 49:             config.MapODataServiceRoute("odata", "odata", builder.GetEdmModel());
        Line 50:         }
        Line 51:     }
        
        Source File: C:\Users\YowkoTsai\Documents\Visual Studio 2017\Projects\TestNLog\TestOData3\App_Start\WebApiConfig.cs    Line: 49
        ```

    * 錯誤截圖

        ![1keynotdefine](https://user-images.githubusercontent.com/3851540/31581624-cd165282-b1a2-11e7-87ee-941ef068b8de.png)

    * 解決方式

        > 在出現問題的 class 上，將 table 的 pk property 加上 `[Key]` 並引用 `using System.ComponentModel.DataAnnotations;`

        ![2solve](https://user-images.githubusercontent.com/3851540/31581625-cd3f7fe0-b1a2-11e7-80bc-d8bec6456e57.png)

## 實際效果

* 修改前

    ![3before](https://user-images.githubusercontent.com/3851540/31581626-cd69807e-b1a2-11e7-96f1-7504aadd792b.png)

* 修改後

    ![4after](https://user-images.githubusercontent.com/3851540/31581627-cd915ee6-b1a2-11e7-935c-b4770f346d52.png)

看得出回傳的 json 結果已經長得不同

## 心得

事實上 OData v3 使用起來並沒有什麼問題，只是想要為 OData 加上 swagger，但 swagger 套件卻只支援 OData v4 ，所以你知道的....順手紀錄一下升級步驟囉，重點還是後面的 OData swagger 哈哈

## 參考資訊

1. [快速搭建WEB API ODATA V4服務端](http://www.debugrun.com/a/vjJZcsL.html)
