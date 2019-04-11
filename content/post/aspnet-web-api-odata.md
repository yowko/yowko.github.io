---
title: "使用 ASP.NET Web API 建立 OData 服務"
date: 2017-10-12T00:50:00+08:00
lastmod: 2018-09-27T00:50:39+08:00
draft: false
tags: ["ASP.NET Web API","EntityFramework","OData"]
slug: "aspnet-web-api-odata"
aliases:
    - /2017/10/aspnet-web-api-odata.html
---
# 使用 ASP.NET Web API 建立 OData 服務
OData 從 2013 年首次出現後就吸引不少技術人員目光，剛好當時專案允許立馬就嘗試，以當時的專案需要同時提供 API 給 WebSite、iOS 及 Android，同樣的資料內容在不同畫面上會因為裝置大小或是版面問題而需要不同欄位，透過 OData 來處理這樣的需求確實方便不少。

最近則是因為專案時程因素，所以預計先長個 OData API 讓前後端的資料可以先銜接起來，日後如果需要調整再來處理(據經驗，通常這樣的規劃在系統上線後就會變成系統穩定就不用改了XD)，無論如何還是來紀錄一下如何建立 OData 服務吧

## 基本環境

1.  Visual Studio 2017
2.  .NET Framework 4.6.2
3.  ASP.NET Web API 2.2
4.  OData v4
5.  Entity Framework 6.1.3


## 建立 ASP.NET Web API 專案

*   建立專案 --> Web --> ASP.NET Web Application

    ![1webapi](https://user-images.githubusercontent.com/3851540/31453717-b0252cee-aee5-11e7-810e-31e194b3514b.png)

*   ASP.NET Empty template --> Web API

    ![2empty](https://user-images.githubusercontent.com/3851540/31453718-b06628ac-aee5-11e7-8946-482c509f167d.png)

## 選項一：使用 scaffolding 建立 OData 服務

1.  Controllers 資料夾上按右鍵 --> Add --> Controller...

    ![3efcontroller](https://user-images.githubusercontent.com/3851540/31453719-b092ea54-aee5-11e7-9632-3e99e40373e8.png)

2.  Controller --> Web API 2 OData v3 Controller with actions,using Entity Framework

    ![4template](https://user-images.githubusercontent.com/3851540/31453721-b0bf7ae2-aee5-11e7-9188-9654aee9e4b5.png)

3.  指定 `Model` 、 context 與 Controller name

    ![5controller](https://user-images.githubusercontent.com/3851540/31453722-b0eb474e-aee5-11e7-8345-a1c8006d06c1.png)

*   需要注意的是使用 scaffolding 來建立 OData 的做法僅限 OData v3 版本，`OData v4 並不支援`
*   過程中會自動安裝以下套件

    *   `Microsoft.AspNet.WebApi.OData 5.3.1`

        > 2017/10/10 最新版為 5.7.0

    *   `Microsoft.Data.Edm 5.6.0`

        > 2017/10/10 最新版為 5.8.3

    *   `Microsoft.Data.OData 5.6.0`

        > 2017/10/10 最新版為 5.8.3

    *   `System.Spatial 5.6.0`

        > 2017/10/10 最新版為 5.8.3

## 選項二：手動建立 OData 服務

*   以下操作需先完成 EntityFramework 連線 DB 設定
    *   Models --> Add --> ADO.NET Enityt Data Model

        ![10adoentity](https://user-images.githubusercontent.com/3851540/31453728-b2076004-aee5-11e7-8c76-67654bd72e14.png)

    *   Code First from data base

        ![11datamodel](https://user-images.githubusercontent.com/3851540/31453729-b23156de-aee5-11e7-9349-b651aec2a1e4.png)

    *   Choose Data Connection

        ![12dataconnection](https://user-images.githubusercontent.com/3851540/31453730-b259f472-aee5-11e7-97b6-2ac11b65be9c.png)

    *   Choose Data Objects

        ![13dataobject](https://user-images.githubusercontent.com/3851540/31453731-b28609a4-aee5-11e7-95ac-5616401ee283.png)

1.  安裝 `Microsoft.AspNet.Odata` 套件

    > 注意這邊的 `Microsoft.AspNet.Odata` 與上面使用的 `Microsoft.AspNet.WebApi.OData` 不同

    >* `Microsoft.AspNet.Odata` 是 OData v4 用
        
    >       ![6odatav4](https://user-images.githubusercontent.com/3851540/31453723-b115c988-aee5-11e7-8af5-19322b41fffd.png)

    >*   `Microsoft.AspNet.WebApi.OData` 則是 OData v1-v3 用
        
    >       ![7odatav3](https://user-images.githubusercontent.com/3851540/31453725-b13fa8ac-aee5-11e7-8e7e-a54f7934add0.png)

    *   使用 `Package Manager Console`

        > `Install-Package Microsoft.AspNet.Odata`

        ![8packageocnsole](https://user-images.githubusercontent.com/3851540/31453726-b17e0106-aee5-11e7-871a-a2a4d236b820.png)

    *   使用 `NuGet Package Manager`

        ![9packagemanager](https://user-images.githubusercontent.com/3851540/31453727-b1c37a92-aee5-11e7-9dde-9abe6f2b7ebc.png)

    *   如果安裝很久出現錯誤或是遲遲無法完成安裝，可以改用 VS2015 安裝看看，我卡了好久改用 VS2015 才成功

2.  建立 OData controller

    > 以測試 db - Northwind 為例

    *   建立 `ShippersController`

        > 加入 `ShippersController.cs`

        ```cs
        public class ShippersController : ODataController
        {
        }
        ```

    *   透過 EntityFramework 檢查資料

        ```CS
        NorthwindEntities db = new NorthwindEntities();
        private bool ShippersExists(int key)
        {
            return db.Shippers.Any(p => p.ShipperID == key);
        }
        ```

    *   加上 Dispose

        ```cs
        protected override void Dispose(bool disposing)
        {
            db.Dispose();
            base.Dispose(disposing);
        }
        ```

    *   完整程式碼

        ```cs
        using System.Linq;
        using System.Web.OData;
        using TestOData.Models;
        
        namespace TestOData.Controllers
        {
            public class ShippersController: ODataController
            {
                NorthwindEntities db = new NorthwindEntities();
                private bool ShippersExists(int key)
                {
                    return db.Shippers.Any(p => p.ShipperID == key);
                }
                protected override void Dispose(bool disposing)
                {
                    db.Dispose();
                    base.Dispose(disposing);
                }
            }
        }
        ```

3.  註冊 OData 服務

    *   開啟 `App_Start/WebApiConfig.cs`
    *   註冊 OData routeing

        > 將新增的程式碼加至 Register 最後

        ```cs
        public static void Register(HttpConfiguration config)
        {
            // New code:
            ODataModelBuilder builder = new ODataConventionModelBuilder();
            builder.EntitySet<Shippers>("Shippers");
            config.MapODataServiceRoute(
                routeName: "ODataRoute",
                routePrefix: "odata",
                model: builder.GetEdmModel());
        }
        ```

    *   完整程式碼

        ```cs
        using System.Net.Http.Formatting;
        using System.Web.Http;
        using System.Web.OData.Builder;
        using System.Web.OData.Extensions;
        using TestOData.Models;
        
        namespace TestOData
        {
            public static class WebApiConfig
            {
                public static void Register(HttpConfiguration config)
                {
                    // Web API configuration and services
                    config.Formatters.Clear();
                    config.Formatters.Add(new JsonMediaTypeFormatter());
                    // Web API routes
                    config.MapHttpAttributeRoutes();
                                config.Routes.MapHttpRoute(
                        name: "DefaultApi",
                        routeTemplate: "api/{controller}/{id}",
                        defaults: new { id = RouteParameter.Optional }
                    );
                                // New code:
                    ODataModelBuilder builder = new ODataConventionModelBuilder();
                    builder.EntitySet<Shippers>("Shippers");
                    config.MapODataServiceRoute(
                        routeName: "ODataRoute",
                        routePrefix: "odata",
                        model: builder.GetEdmModel());
                }
            }
        }
        ```

4.  加入 CRUD

    *   Get - Read

        ```cs
        [EnableQuery]
        public IQueryable<Shippers> Get()
        {
            return db.Shippers;
        }
        [EnableQuery]
        public SingleResult<Shippers> Get([FromODataUri] int key)
        {
            IQueryable<Shippers> result = db.Shippers.Where(p => p.ShipperID == key);
            return SingleResult.Create(result);
        }
        ```

    *   Post - Create

        ```cs
        public async Task<IHttpActionResult> Post(Shippers shippers)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }
            db.Shippers.Add(shippers);
            await db.SaveChangesAsync();
            return Created(shippers);
        }
        ```

    *   Put/Patch - Update

        ```cs
        public async Task<IHttpActionResult> Patch([FromODataUri] int key, Delta<Shippers> shippers)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }
            var entity = await db.Shippers.FindAsync(key);
            if (entity == null)
            {
                return NotFound();
            }
            shippers.Patch(entity);
            try
            {
                await db.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException)
            {
                if (!ShippersExists(key))
                {
                    return NotFound();
                }
                else
                {
                    throw;
                }
            }
            return Updated(entity);
        }

        public async Task<IHttpActionResult> Put([FromODataUri] int key, Shippers update)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }
            if (key != update.ShipperID)
            {
                return BadRequest();
            }
            db.Entry(update).State = EntityState.Modified;
            try
            {
                await db.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException)
            {
                if (!ShippersExists(key))
                {
                    return NotFound();
                }
                else
                {
                    throw;
                }
            }
            return Updated(update);
        }
        ```

    *   Delete - Delete

        ```cs
        public async Task<IHttpActionResult> Delete([FromODataUri] int key)
        {
            var product = await db.Shippers.FindAsync(key);
            if (product == null)
            {
                return NotFound();
            }
            db.Shippers.Remove(product);
            await db.SaveChangesAsync();
            return StatusCode(HttpStatusCode.NoContent);
        }
        ```

    *   完整程式碼

        ```cs
        using System.Data.Entity;
        using System.Data.Entity.Infrastructure;
        using System.Linq;
        using System.Net;
        using System.Threading.Tasks;
        using System.Web.Http;
        using System.Web.OData;
        using TestOData.Models;
        
        namespace TestOData.Controllers
        {
            public class ShippersController: ODataController
            {
                NorthwindEntities db = new NorthwindEntities();
                [EnableQuery]
                public IQueryable<Shippers> Get()
                {
                    return db.Shippers;
                }
                [EnableQuery]
                public SingleResult<Shippers> Get([FromODataUri] int key)
                {
                    IQueryable<Shippers> result = db.Shippers.Where(p => p.ShipperID == key);
                    return SingleResult.Create(result);
                }
                public async Task<IHttpActionResult> Post(Shippers shippers)
                {
                    if (!ModelState.IsValid)
                    {
                        return BadRequest(ModelState);
                    }
                    db.Shippers.Add(shippers);
                    await db.SaveChangesAsync();
                    return Created(shippers);
                }
                public async Task<IHttpActionResult> Patch([FromODataUri] int key, Delta<Shippers> shippers)
                {
                    if (!ModelState.IsValid)
                    {
                        return BadRequest(ModelState);
                    }
                    var entity = await db.Shippers.FindAsync(key);
                    if (entity == null)
                    {
                        return NotFound();
                    }
                    shippers.Patch(entity);
                    try
                    {
                        await db.SaveChangesAsync();
                    }
                    catch (DbUpdateConcurrencyException)
                    {
                        if (!ShippersExists(key))
                        {
                            return NotFound();
                        }
                        else
                        {
                            throw;
                        }
                    }
                    return Updated(entity);
                }
                public async Task<IHttpActionResult> Put([FromODataUri] int key, Shippers update)
                {
                    if (!ModelState.IsValid)
                    {
                        return BadRequest(ModelState);
                    }
                    if (key != update.ShipperID)
                    {
                        return BadRequest();
                    }
                    db.Entry(update).State = EntityState.Modified;
                    try
                    {
                        await db.SaveChangesAsync();
                    }
                    catch (DbUpdateConcurrencyException)
                    {
                        if (!ShippersExists(key))
                        {
                            return NotFound();
                        }
                        else
                        {
                            throw;
                        }
                    }
                    return Updated(update);
                }
                public async Task<IHttpActionResult> Delete([FromODataUri] int key)
                {
                    var product = await db.Shippers.FindAsync(key);
                    if (product == null)
                    {
                        return NotFound();
                    }
                    db.Shippers.Remove(product);
                    await db.SaveChangesAsync();
                    return StatusCode(HttpStatusCode.NoContent);
                }
                private bool ShippersExists(int key)
                {
                    return db.Shippers.Any(p => p.ShipperID == key);
                }
                protected override void Dispose(bool disposing)
                {
                    db.Dispose();
                    base.Dispose(disposing);
                }
            }
        }
        ```

## 實際效果

*   Get

    > `http://localhost:6600/odata/Shippers`

    ![14get](https://user-images.githubusercontent.com/3851540/31453733-b2b27156-aee5-11e7-9df0-ada2ea6586a1.png)

*   Post

    > `http://localhost:6600/odata/Shippers`

    ![15post](https://user-images.githubusercontent.com/3851540/31453734-b2e21582-aee5-11e7-827f-809e0de5dd10.png)

*   Patch

    > `http://localhost:6600/odata/Shippers(5)`

    ![16patch](https://user-images.githubusercontent.com/3851540/31453735-b30c2368-aee5-11e7-8d5c-e77248f9cfcf.png)

*   Put

    > `http://localhost:6600/odata/Shippers(5)`

    ![17put](https://user-images.githubusercontent.com/3851540/31453736-b3357b28-aee5-11e7-92dd-640860adf9d9.png)

*   Delete

    > `http://localhost:6600/odata/Shippers(5)`

    ![18delete](https://user-images.githubusercontent.com/3851540/31453737-b3627128-aee5-11e7-8aa0-17dd2c548279.png)

## 心得

這篇筆記花了好幾天才寫完，陸陸續續遇到不少問題，也卡關了好幾次，但終究是搞定了~~ 呼

雖然不知道原因，但可以確定的是 OData v4 有一些跟之前版本不同的 breaking change 讓使用過之前版本的工程師佔不到便宜，還好趁著這次紀錄建立過程釐清不少相關設定，降低系統上線時因為不熟悉設定造成問題的機率

僅管對於設定有比較了解一些，但對於相關原理及進階使用方式的掌握度還是有待加強，相信隨著專案的進行會一步一步更加清晰，到時有什麼值得紀錄介紹的再跟大家分享

# 參考資訊

1.  [OData in ASP.NET Web API](https://docs.microsoft.com/en-us/aspnet/web-api/overview/odata-support-in-aspnet-web-api/)
2.  [OData Error: The query specified in the URI is not valid. The property cannot be used in the query option](https://stackoverflow.com/questions/39515218/odata-error-the-query-specified-in-the-uri-is-not-valid-the-property-cannot-be)
3.  [GETTING STARTED WITH WEB API AND ODATA V4 PART 1](https://damienbod.com/2014/06/10/getting-started-with-web-api-and-odata-v4/)
