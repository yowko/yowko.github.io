---
title: "關於 GetType 的一些事"
date: 2017-07-30T22:30:00+08:00
lastmod: 2018-09-24T22:30:15+08:00
draft: false
tags: ["C#"]
slug: "gettype"
aliases:
    - /2017/07/gettype.html
---
# 關於 GetType 的一些事
之前在 [LINQ to Objects VS LINQ to Entities](https://blog.yowko.com/2017/07/linq-to-objects-vs-linq-to-entities.html) 中提到 LINQ to Objects 是一系列 `IEnumerable` 及 `IEnumerable<T>` 的擴充方法，而 LINQ to Entities 是一系統 `IQueryable` 及 `IQueryable<T>` 的擴充方法，透過物件繼承的型別可以很容易區分出 `LINQ to Objects` 或是 `LINQ to Entities`，只是該如何得知物件的繼承內容呢？

以下內容是 黃忠成老師 在 `LINQ - 強者之道` 課程中所使用的小技巧，我覺得很有幫助，紀錄一下

## 關於 GetType 方法

> 官方文件可以參考 [Object.GetType Method](https://docs.microsoft.com/en-us/dotnet/api/system.object.gettype)

1.  `Object` class 所實作的方法

    > 也就是說適用於所有 .NET Framework 物件

2.  GetType 方法回傳的五種 .NET Framework type
    *   Classes

        > 繼承自 `System.Object`

    *   Value types

        > 繼承自 `System.ValueType`

    *   Interfaces

        > 繼承自 `System.Object`，從 .NET Framework 2.0 開始出現

    *   Enumerations

        > 繼承自 `System.Enum`

    *   Delegates

        > 繼承自 `System.MulticastDelegate`

## 取得物件的基底型別

> `object.GetType().BaseType`

得知基底型別後，就可以進一步了解基底型別是否有實作其他細節

![1gettype](https://user-images.githubusercontent.com/3851540/28754269-d40109b2-7574-11e7-927e-abd231f0d90d.png)

## 取得物件實作的 Interface

> `object.GetType().GetInterfaces()`

可以直接得知物件實作哪些 Interface，進而了解基於 Interface 所擁有的功能

![2getinterfaces](https://user-images.githubusercontent.com/3851540/28754270-d42591b0-7574-11e7-9c8b-8da94ed0c7ae.png)

## 心得

之前使用 `GetType` 都是為了 reflection，剛好這次 黃忠成老師示範了如何使用 `GetType` 來確認物件進行 linq 操作時所使用的是 `LINQ to Objects` 或是 `LINQ to Entities`，也讓我學到如何在使用 linq 操作物件時更明確地知道我使用的技術細節，再次感謝 黃忠成老師

# 參考資訊

1.  [LINQ to Objects VS LINQ to Entities](https://blog.yowko.com/2017/07/linq-to-objects-vs-linq-to-entities.html)
2.  [Object.GetType Method](https://docs.microsoft.com/en-us/dotnet/api/system.object.gettype)
