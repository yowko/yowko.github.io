---
title: "ClickHouse 在彙總資料時的效能優勢"
date: 2024-01-03T00:30:00+08:00
lastmod: 2024-01-03T00:30:31+08:00
draft: false
tags: ["docker","csharp","clickhouse","mysql"]
slug: "clickhouse-aggregate-query-performance"
---

## ClickHouse 在彙總資料時的效能優勢

前提：雖然本篇筆記中使用了 MySql 做為範例，但主要是因為目前團隊使用 MySql，我對 MySql 相同熟悉點，而不是為了踩 MySql 捧 ClickHouse，而是為了突顯在不同目的與情境下，選擇更適合的技術會比 performance tuning 更有效率

一般系統在正常運作下，不免會有將資料彙總這類報表的需求，例如：統計每個月的銷售量、每個月的訂單數量、每個月的訂單金額等等，這時候就會需要用到 `group by` 這個語法，這類查詢方式很容易讓傳統 OLTP 以 row 為儲存基準的 DB 陷入效能瓶頸，本篇筆記就是要呈現 ClickHouse 在彙總資料時的效能優勢

本篇筆記用到基本環境，詳細內容可以參考之前筆記 [C# 如何新增資料至 ClickHouse](/csharp-insert-clickhouse/) 與 [C# 如何快速新增大量資料至 MySQL](/csharp-mysql-bulk-insert/)

## 基本環境說明

1. macOS Sonoma 14.2.1 (Apple M2 Pro)
2. OrbStack Version 1.2.0 (16496)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.2
5. Container Images

    - clickhouse/clickhouse-server:23.11.3.23
    - mysql:8.2

6. NuGet Library

    - Bogus 35.3.0
    - BenchmarkDotNet 0.13.11
    - ClickHouse.Ado 2.0.2.2
    - ClickHouse.Client 6.8.1
    - MySqlConnector 2.3.3
    - CsvHelper 30.0.1

7. docker-compose.yml

    {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "docker-compose.yml">}}

8. ClickHouse DDL

    {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "clickhouse-ddl.sql">}}

9. MySql DDL

    {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "mysql-ddl.sql">}}

10. 測試資料

    > 產生一億筆資料

    {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "DataGenProgram.cs">}}

## 實際比較

- 情境說明
    - 以 OrderType 分組，計算每個 OrderType 的總金額

        ```sql
        select order_type,sum(amount) from test.orders group by order_type; 
        ```

    - 以 ProductId 分組，計算每個 ProductId 的總金額

        ```sql
        select product_id,sum(amount) from test.orders group by product_id;
        ```

    - 指定 OrderDate 區間計算總金額

        ```sql
        SELECT sum(amount)
        FROM test.orders
        WHERE order_date BETWEEN '2023-11-01' AND '2023-11-30';
        ```

    - 指定 OrderDate 區間計算總金額，並以 OrderType 分組

        ```sql
        SELECT order_type, sum(amount)
        FROM test.orders
        WHERE order_date BETWEEN '2022-04-01' AND '2022-04-30'
        GROUP BY order_type;
        ```

- 效能比較

    - 程式碼

        {{<gist yowko d209d6aad7432f59a99e5c38377697f5 "QueryBenchmarkProgram.cs">}}

    - 比較結果

        ![1benchmark](https://github.com/yowko/picsbed/assets/3851540/4f8efdd4-711b-47e2-864e-73794f08ac68)

## 心得

1. ClickHouse 與 MySql 彙總出來的金額有落差，我快速確認過較小的資料就有這個傾向，看起來是對於小數點後的資料處理方式不同，但這不是本篇筆記的重點，所以就不深入研究了，但如果後續要實際應用還是需要解決，畢竟錢的問題不能含糊
2. 不管是 ClickHouse 或是 MySql 在 table 的定義、index 的調整或是 table engine 的選擇，甚至是 query script 的寫法上都有不小優化空間，但還是想用這個例子來呈現不同技術在相同情境下的效能差異
3. 單就本筆記的簡單範例看來，相同查詢在 ClickHouse 與 MySql 有接近 200 倍的效能差異

## 參考資訊

1. [What Is ClickHouse?](https://clickhouse.com/docs/en/intro)
2. [C# 如何新增資料至 ClickHouse](/csharp-insert-clickhouse/)
3. [C# 如何快速新增大量資料至 MySQL](/csharp-mysql-bulk-insert/)
