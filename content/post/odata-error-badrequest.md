---
title: "OData 無法使用 $count, $orderby, $select, $top, $expand, $filter"
date: 2017-10-14T01:10:00+08:00
lastmod: 2018-09-27T01:10:05+08:00
draft: false
tags: ["ASP.NET Web API","Debug","OData"]
slug: "odata-error-badrequest"
aliases:
    - /2017/10/odata-error-badrequest.html
---
# OData 無法使用 $count, $orderby, $select, $top, $expand, $filter
同事回報在介接 OData API 時發現無法使用 `$filter` 來過濾資料，經一步測試發現不僅是 `$filter` 無法使用，OData 其他運算子都無法正常運作，來看看該如何處置吧

## 錯誤訊息 1

*   訊息內容

    ```
    The query specified in the URI is not valid. The property 'Id' cannot be used in the $filter query option.
    ```

*   錯誤截圖

    ![1filter](https://user-images.githubusercontent.com/3851540/31557497-908ef008-b07b-11e7-9dee-a5bc0da69fae.png)

## 錯誤訊息 1

*   訊息內容

    ```
    The query specified in the URI is not valid. The limit of '0' for Top query has been exceeded. The value from the incoming request is '1'.
    ```

*   錯誤截圖
    
    ![2top](https://user-images.githubusercontent.com/3851540/31557498-90b940a6-b07b-11e7-91f2-d78c98a9669f.png)


## 問題發生原因

```
Now the default setting for WebAPI OData is : client can't apply $count, $orderby, $select, $top, $expand, $filter in the query, query like localhost\odata\Customers?$orderby=Name will failed as BadRequest, because all properties are not sort-able by default, this is a breaking change in 6.0.0,
```

詳細內容請參考 [13.1 Model Bound Attributes](http://odata.github.io/WebApi/#13-01-modelbound-attribute)

    > 這是 6.0.0 的異動內容

## 解決方式

1.  修改全域查詢設定
    *   在 `WebApiConfig` 的 `Register` 中加上下列設定

        ```
        config.Count().Filter().OrderBy().Expand().Select().MaxTop(null);
        ```

    *   修改範例

        ```cs
        ODataConventionModelBuilder builder = new ODataConventionModelBuilder();
        config.Count().Filter().OrderBy().Expand().Select().MaxTop(null);
        builder.EntitySet<BU>("BU");
        config.MapODataServiceRoute(
                routeName: "ODataRoute",
                routePrefix: "odata",
                model: builder.GetEdmModel()
        );
        ```

2.  僅修改需要的 Entity 設定
    *   filter id

        > 在 builder.EntitySet 後加上 `builder.EntityType<BU>().Filter("Id");`

        ```cs
        ODataConventionModelBuilder builder = new ODataConventionModelBuilder();
        builder.EntitySet<BU>("BU");
        builder.EntityType<BU>().Filter("Id");
        builder.GetEdmModel());
        config.MapODataServiceRoute(
            routeName: "ODataRoute",
            routePrefix: "odata",
            model: builder.GetEdmModel()
        );
        ```

    *   top

        > 在 builder.EntitySet 後加上 `builder.EntityType<BU>().Page(100, 100);`

        ```cs
        ODataConventionModelBuilder builder = new ODataConventionModelBuilder();
        builder.EntitySet<BU>("BU");
        builder.EntityType<BU>().Page(100, 100);
        builder.GetEdmModel());
        config.MapODataServiceRoute(
            routeName: "ODataRoute",
            routePrefix: "odata",
            model: builder.GetEdmModel()
        );
        ```

## 實際效果

![3filterok](https://user-images.githubusercontent.com/3851540/31557499-90e6fa28-b07b-11e7-873f-b8f8b294431b.png)

![4topok](https://user-images.githubusercontent.com/3851540/31557500-910e864c-b07b-11e7-9d2b-ce3b21aeb8e8.png)

## 心得

我看到問題發生原因是 breaking change 時，我好傻眼，原來之前一直覺得 OData 舊版本使用者討不到便宜的原因是出在這XD，但憑心而論這實在是自己的問題，誰叫我自己不看 release notes 就直接開工，自以為用過就有恃無恐，吃到苦頭了吧@@"

# 參考資訊

1.  [OData Error: The query specified in the URI is not valid. The property cannot be used in the query option](https://stackoverflow.com/questions/39515218/odata-error-the-query-specified-in-the-uri-is-not-valid-the-property-cannot-be)
2.  [WebApi OData The limit of '0' for Top query has been exceeded, even after setting MaxTop](https://stackoverflow.com/questions/41849692/webapi-odata-the-limit-of-0-for-top-query-has-been-exceeded-even-after-settin)
3.  [13.1 Model Bound Attributes](http://odata.github.io/WebApi/#13-01-modelbound-attribute)
