---
title: "LINQ to Objects VS LINQ to Entities"
date: 2017-07-17T23:20:00+08:00
lastmod: 2017-07-17T23:20:51+08:00
draft: false
tags: ["C#"]
slug: "linq-to-objects-vs-linq-to-entities"
aliases:
    - /2017/07/linq-to-objects-vs-linq-to-entities.html
---
# LINQ to Objects VS LINQ to Entities
前幾天去參加 黃忠成 老師的 LINQ - 強者之道時，聽到忠成老師說 LINQ 有兩種行為：LINQ to Objects 與 LINQ to Entities。What！！用那麼久 LINQ，原來我根本不懂 LINQ，實在太令人汗顏了。還好我有去上課，後知後覺地學習也是進步的一種管道，趕緊來了解一下其中差異

## LINQ 概觀

![LINQ overview](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/media/dpue-linqtoadonetoverview-bpuedev11.gif)圖片出處 [LINQ and ADO.NET](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/linq-and-ado-net)

## LINQ to Objects

Visual Basic 與 C# 依據語言特性而在 LINQ to Objects 的行為上有些不同(以下內容針對 C#)

1.  [LINQ to Objects (Visual Basic)](https://docs.microsoft.com/en-us/dotnet/visual-basic/programming-guide/concepts/linq/linq-to-objects)
2.  [LINQ to Objects (C#)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects)


    *   是一系統針對 `IEnumerable` 及 `IEnumerable<T>` 的擴充方法
    *   主要針對 enumerable 集合進行查詢，包括 List、Array、Dictionary、Stack、SortedList、String
    *   不使用像是 LINQ to SQL or LINQ to XML 這類的中繼 LINQ provider 或是 API
    *   可以呼叫自訂 delegate
    *   進行 in-memory 的查詢


## LINQ to Entities

*   LINQ to Entities 是一種 LINQ provider，提供一系統針對 `IQueryable` 及 `IQueryable<T>` 的擴充方法
*   LINQ to Entities 提供的擴充方式是由 expression tree 建立的
*   provider 會將 expression tree 內容轉譯為適合的 SQL 語法進行查詢
*   需要建立 ObjectContext instance
*   預設使用延遲查詢 (Deferred query)


    *   可透過使用 `ToList`, `ToArray`, `ToLookup`, `ToDictionary` 立即執行查詢
    *   取固定值 `Average`, `Count`, `First`, `Max` 方法則是立即執行查詢

*   可強制使用 LINQ to Objects


    *   使用 `AsEnumerable()` 可以將 LINQ to Entities 轉為 LINQ to Objects
    *   留意效能瓶頸


        *   LINQ to Entities

            > `Users.Where(u => u.Name=="Yowko")`

            *   會將過濾條件轉換為 sql 語法，並在 db 完成過濾後回傳

        *   LINQ to Objects

            > `Users.AsEnumerable().Where(u => u.Name=="Yowko")`

            *   會將 Users 資料全數取回後再 memory 進行過濾

    *   兩者在小資料集時差異不大，但大資料集光 network io 就差很多了

## 心得

不知道是不是大家都知道兩者明顯差異，網路上的比較文章很少。

經過認真 k 了相關文件，發現原來我清楚兩者差異還曾經使用來調校程式效能，只是不知道兩者原來各有專有名詞 ~~~ 真冏

# 參考資訊

1.  [linq to entities vs linq to objects - are they the same?](https://stackoverflow.com/questions/7192040/linq-to-entities-vs-linq-to-objects-are-they-the-same)
2.  [LINQ to Objects](https://msdn.microsoft.com/en-us/library/bb397919.aspx)
3.  [LINQ to Objects (Visual Basic)](https://docs.microsoft.com/en-us/dotnet/visual-basic/programming-guide/concepts/linq/linq-to-objects)
4.  [LINQ to Objects (C#)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects)
