---
title: "如何為 SQL SERVER 建立資料庫版控"
date: 2017-04-09T21:21:00+08:00
lastmod: 2018-09-16T00:21:18+08:00
draft: false
tags: ["SQL Server","Visual Studio"]
slug: "sql-server-version-control"
aliases:
    - /2017/04/sql-server-data-tools.html
---
# 如何為 SQL SERVER 建立資料庫版控
程式碼版控的重要性對於開發人員不言可喻，但很常遇到的問題是資料庫相依：程式有完整版控但每個版本對應的資料庫 schema 或是資料可能無法配合，導致程式在版本轉換時還是非常依賴人力的介入

針對這個問題，非常感謝微軟有認真地設身處地想要解決開發人員的困境，微軟透過 SQL Server Data Tools (SSDT) 來輔助開發人員處理資料庫的版本控管

透過 SQL Server Data Tools (SSDT) 可以將 SQL SERVER 中的各種資料庫物件轉換為程式碼，處理方式與 Object Relational Mapping(ORM) 相似，經過轉換就可以把資料庫當做一般程式碼來進行版控了

## 如何開始

SSDT 是基於 Visual Studio 的工具，如果在安裝 SSDT 前未安裝 Visual Studio ，SSDT 會自動安裝一個 Visual Studio 整合模式的版本，但為了發揮 SSDT 強大功能還是建議安裝 Visual Studio Community 以上版本

SSDT 可以前往 [下載 SQL Server Data Tools (SSDT)](https://msdn.microsoft.com/library/mt204009.aspx) 下載並安裝

## 建立資料庫專案

> 本文的範例使用 Visual Studio 2015 ，不同版本操作畫面可能略有不同

1.  Installed 專案範本
2.  SQL Server
3.  SQL Server Database Project
4.  Project Name
5.  Ok --> 建立資料庫專案


    ![1dbproj](https://cloud.githubusercontent.com/assets/3851540/24837568/8300bc50-1d69-11e7-9cd9-d6039de20bd7.png)

## 建立資料庫物件

1.  匯入資料庫(擇一即可)

    *   專案右鍵 --> Import --> Database...
        
        ![2importdb](https://cloud.githubusercontent.com/assets/3851540/24837569/8302c66c-1d69-11e7-99b5-fa459ac65702.png)

    *   Visual Studio 主選單中的 Project --> Import --> Database...
        
        ![10import](https://cloud.githubusercontent.com/assets/3851540/24837567/8300722c-1d69-11e7-92d1-8f4166965f5d.png)

2.  選擇資料庫連線

    *   Select Connection...

        ![3selectconnection](https://cloud.githubusercontent.com/assets/3851540/24837570/8321b982-1d69-11e7-9b2c-c409403588a3.png)

    *   使用之前連線資訊 or 瀏覽並填寫連線資訊

        ![4browser](https://cloud.githubusercontent.com/assets/3851540/24837571/8322802e-1d69-11e7-8a4c-72c40a15fc7e.png)

        ![5fillin](https://cloud.githubusercontent.com/assets/3851540/24837572/8324cf00-1d69-11e7-9967-e5b541034233.png)

    *   開始匯入

        ![6start](https://cloud.githubusercontent.com/assets/3851540/24837573/832799a6-1d69-11e7-9ab5-f9f14e9262f3.png)

        ![7importing](https://cloud.githubusercontent.com/assets/3851540/24837564/82fcc78a-1d69-11e7-9424-38097a87590d.png)

        ![8imported](https://cloud.githubusercontent.com/assets/3851540/24837566/82fe0d5c-1d69-11e7-9c49-c37b89137a96.png)

## 完整資料庫物件專案

![9result](https://cloud.githubusercontent.com/assets/3851540/24837565/82fdd846-1d69-11e7-9828-b2ea38ddbf14.png)

由完成匯入的結果，可以看到資料庫物件已被轉成 .cs 程式碼儲存，其中不僅包含了基本的 table、function、Stored Procedure，甚至連 trigger、User Defined Type 都有 ，如此一來我們就可以讓不同開發階段甚至是每個開發人員都能擁有專屬的 db 而不會影響其他人，直到功能確認開發完畢再進行 merge 以及 schema change 即可

## 心得

SSDT 功能如同其他微軟工具，持續地改善演進中，今天初步分享以往專案平行開發時用來避免因為 db schema 不一致而常常需要溝通協調的問題，有機會會再介紹其他功能及用法

# 參考資訊
1.  [SQL Server Data Tools](https://msdn.microsoft.com/library/hh272686%28v=vs.103%29.aspx)
2.  [下載 SQL Server Data Tools (SSDT)](https://msdn.microsoft.com/library/mt204009.aspx)
