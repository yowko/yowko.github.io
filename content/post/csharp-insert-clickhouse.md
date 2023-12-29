---
title: "C# 如何新增資料至 ClickHouse"
date: 2023-12-27T00:30:00+08:00
lastmod: 2023-12-27T00:30:31+08:00
draft: false
tags: ["docker","csharp","clickhouse"]
slug: "csharp-insert-clickhouse"
---

## C# 如何新增資料至 ClickHouse

最近在評估導入 ClickHouse 的可行性，首先除了測試環境建立之外，最重要的就是要能夠透過 C# 來新增資料，所以今天就來紀錄如何透過 C# 來新增資料至 ClickHouse 並且比較不同 library 的優缺點

以下關於 ClickHouse 的介紹彙整自 ChatGPT3.5 與 Bard ，另外 [What Is ClickHouse?](https://clickhouse.com/docs/en/intro) 也提供了更詳細的說明

- ClickHouse 是由俄羅斯公司 Yandex 開發並 open source 的列式數據庫，它具有以下特色：

    1. 列式存儲架構： ClickHouse使用列式存儲架構，將相同類型的數據存儲在一起，這種方式非常適合分析性查詢。列式存儲使得只擷取需要的欄位變得更加高效，這對於大規模的數據分析操作非常有利。

    2. 高效的壓縮算法： ClickHouse使用了多種高效的壓縮算法，減少數據在磁盤上的存儲空間，同時在查詢時能夠更迅速地解壓縮數據。這有助於節省硬件成本，提高系統性能。

    3. 支援實時數據更新： 雖然ClickHouse主要用於分析型查詢，但它也支援實時數據更新，能夠應對需要即時反饋的場景，擴展了其應用範圍。

    4. 分散式架構： ClickHouse的分散式架構使其能夠水平擴展，以應對不斷增長的數據量。這種能力對於處理大規模數據非常重要，同時也提高了系統的可靠性和容錯性。

- ClickHouse 適用於以下情境：

    1. 大規模數據分析： ClickHouse最擅長處理大量數據的分析查詢，例如數據採礦、BI（商業智能）分析、日誌分析等。它能夠在秒級別內處理龐大的數據集，提供即時且準確的結果。

    2. 時序數據處理： ClickHouse在處理時序數據（例如日誌、事件時間數據）方面表現出色。其列式存儲和高效的壓縮算法使得它成為時序數據處理的理想選擇。

    3. OLAP查詢： ClickHouse適合執行複雜的OLAP（On-Line Analytical Processing）查詢，支援諸如分組、排序、過濾等操作，為用戶提供靈活的數據分析功能。

- 具體來說，ClickHouse 在以下類型的查詢上具有較好的效能表現：

    - 聚合查詢：ClickHouse 在聚合查詢上具有出色的性能表現，可在秒級內完成百萬級數據的聚合查詢。
    - 分組查詢：ClickHouse 在分組查詢上也具有較好的性能表現，可在秒級內完成百萬級數據的分組查詢。
    - 過濾查詢：ClickHouse 在過濾查詢上也具有較好的性能表現，可在毫秒級內完成百萬級數據的過濾查詢。

總而言之，ClickHouse 以其高效的列式存儲架構、強大的分析能力和分散式架構，成為處理大規模數據分析的優秀選擇，尤其在時序數據處理和OLAP查詢方面表現卓越。

## 基本環境說明

1. macOS Sonoma 14.2.1 (Apple M2 Pro)
2. OrbStack Version 1.2.0 (16496)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.2
5. Container Images

    - clickhouse/clickhouse-server:23.11.3.23

6. NuGet Library

    - Bogus 35.2.0
    - BenchmarkDotNet 0.13.11
    - ClickHouse.Ado 2.0.2.2
    - ClickHouse.Client 6.8.1

7. docker-compose.yml

    {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "docker-compose.yaml">}}

8. clickhouse DDL

    {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "ddl.sql">}}

## 使用方式

`ClickHouse.Client` 是 NuGet 近期下載量較多的 library，`ClickHouse.Ado` 則是發展較久，總下載量較高，都一併試試看

- ClickHouse.Client

    - connection string 支援參數，詳細內容可以參考 [GitHub:DarkWanderer/ClickHouse.Client:Connection string](https://github.com/DarkWanderer/ClickHouse.Client/wiki/Connection-string)

        |Parameter name|Description|Value if omitted|
        |---|---|---|
        |Database|連線的初始資料庫|`default`|
        |Username|Authentication username|`default`|
        |Password|Authentication password||
        |Host|ClickHouse server address|`localhost`|
        |Port|ClickHouse server port|`8123` or `8443` (取決於 `Protocol` and `library version`)|
        |Driver|自 2.0 版本起已棄用|`Binary`|
        |Compression|啟用資料的 GZip 壓縮| 從版本 2.0 開始是 `true`，之前是 `false`|
        |UseSession|允許使用伺服器會話|`false`|
        |SessionId|指定 session ID (string)|random GUID|
        |Timeout|HTTP timeout duration|`120`|
        |Protocol|`HTTP` or `HTTPS`|`http`|
        |UseServerTimezone|對沒有 TZ 規範的日期時間列使用伺服器時區|從版本 6.0 開始是 `true`，之前是 `false`|
        |UseCustomDecimals|使用 `ClickHouseDecimal` classe 來處理 decimal|`false`|

    - 程式碼

        {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "ClickHouseClient.cs">}}

- ClickHouse.Ado

    - connection string 支援參數，詳細內容可以參考 [GitHub:ClickHouse.Test/ClickHouseConnectionSettingsTests/ToStringTests.cs](https://github.com/killwort/ClickHouse-Net/blob/e70be1f7435b8ebb4aea97ccf261a3593590c93b/ClickHouse.Test/ClickHouseConnectionSettingsTests/ToStringTests.cs) 與 [GitHub:ClickHouse.Ado/ClickHouseConnectionSettings.cs](https://github.com/killwort/ClickHouse-Net/blob/e70be1f7435b8ebb4aea97ccf261a3593590c93b/ClickHouse.Ado/ClickHouseConnectionSettings.cs)

        |Parameter name|Default Value|
        |---|---|
        |BufferSize|`4096`|
        |SocketTimeout|`1000`|
        |Host||
        |Port|`9000` or `9440`|
        |Database|`default`|
        |Compress|`false`|
        |Compressor| 需要設定為 `LZ4`|
        |CheckCompressedHash|`true`|
        |User||
        |Password||

    - 程式碼

        {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "ClickHouseAdo.cs">}}

- 完整程式碼

    {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "Program.cs">}}

## 心得

1. 雖然 `ClickHouse.Ado` 推出較早，但完全沒有文件可以參考，只能透過看 source code 或是 test case 來了解用法跟設定，單就這點就不是很理想
2. `ClickHouse.Ado` 沒有提供 release 版本，程式碼沒有經過最佳化，benchmark 也需要額外設定
3. protocol 與 port
   - `ClickHouse.Ado` 使用的 `tcp` 連線，port 是 `9000` or `9440` (tls)
   - `ClickHouse.Client` 使用 `http` 連線，port `8123` or `8443` (https)
4. 我之前使用 `ClickHouse.Client` 有遇到 insert 速度很慢的狀況，後來換成 `ClickHouse.Ado` 有解決問題，原本以為是 protocol 造成的差異；但這次整理做 benchmark 時也沒有遇到，甚至在資料量較多時(100 萬筆)，`ClickHouse.Client 比 ClickHouse.Ado 還快`，所以這點還需要再觀察
5. NuGet 上的另個 library `Octonica.ClickHouseClient` 近期的下載量也很高，但我照著 GitHub 上的 readme 跟 test 嘗試 bulk insert 都無法成功就不加入這次比較了

- 執行速度與 memory 耗用

    ![1benchmark](https://github.com/yowko/picsbed/assets/3851540/6e08ddd3-74a3-443f-b096-f1beaa8a4be2)

完整程式碼請參考：[GitHub:yowko/ClickHouseLibraryTest](https://github.com/yowko/ClickHouseLibraryTest)

## 參考資料

1. [What Is ClickHouse?](https://clickhouse.com/docs/en/intro)
2. [GitHub:DarkWanderer/ClickHouse.Client:Connection string](https://github.com/DarkWanderer/ClickHouse.Client/wiki/Connection-string)
3. [GitHub:ClickHouse.Test/ClickHouseConnectionSettingsTests/ToStringTests.cs](https://github.com/killwort/ClickHouse-Net/blob/e70be1f7435b8ebb4aea97ccf261a3593590c93b/ClickHouse.Test/ClickHouseConnectionSettingsTests/ToStringTests.cs)
4. [GitHub:ClickHouse.Ado/ClickHouseConnectionSettings.cs](https://github.com/killwort/ClickHouse-Net/blob/e70be1f7435b8ebb4aea97ccf261a3593590c93b/ClickHouse.Ado/ClickHouseConnectionSettings.cs)
5. [ClickHouse Server Docker Image](https://hub.docker.com/r/clickhouse/clickhouse-server)
6. [GitHub:yowko/ClickHouseLibraryTest](https://github.com/yowko/ClickHouseLibraryTest)
