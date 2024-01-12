---
title: "為頻繁更新 ClickHouse 資料選擇適合的 Table Engine"
date: 2024-01-12T00:30:00+08:00
lastmod: 2024-01-12T00:30:31+08:00
draft: false
tags: ["docker","csharp","clickhouse"]
slug: "clickhouse-replacingmergetree-collapsingmergetree"
---

## 為頻繁更新 ClickHouse 資料選擇適合的 Table Engine

雖然之前筆記 [新增修改刪除查詢 ClickHouse 資料"](/clickhouse-insert-update-select-delete) 有紀錄到如何更新與刪除資料，但在 ClickHouse 官網文件：[Avoid Mutations](https://clickhouse.com/docs/en/optimize/avoid-mutations) 就明確提到 `ALTER TABLE` (大多 `ALTER TABLE` 語法只支援 `*MergeTree` table)、`DELETE`、`UPDATE` 這類語法會造成資料產生變異版本，使得資料需要重寫而產生大量的寫入行為，因此在 ClickHouse 中不建議使用這類語法，至於資料更新需求則是建議使用其他更適合的 table engine 像是 `ReplacingMergeTree` 或 `CollapsingMergeTree`，所以今天就來紀錄一下 `ReplacingMergeTree` 與 `CollapsingMergeTree` 的使用與差異

## 基本環境說明

1. macOS Sonoma 14.2.1 (Apple M2 Pro)
2. OrbStack Version 1.2.0 (16496)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.2
5. Container Images

    - clickhouse/clickhouse-server:23.11.3.23

6. NuGet Library

    - Bogus 35.3.0
    - ClickHouse.Client 6.8.1

7. docker-compose.yml

    {{<gist yowko 013af07bc26b4b1f7f361cb65a4a5b3f "docker-compose.yaml">}}

## ReplacingMergeTree 與 CollapsingMergeTree 的簡介與使用方式

1. ReplacingMergeTree

    完整內容請參考官網文件：[ReplacingMergeTree](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replacingmergetree)

    - 特色：

        - 會刪除具有相同 `ORDER BY` 欄位的重複資料而非 `PRIMARY KEY`
        - 重複資料刪除僅在 merge 時發生，merge 會在背景執行，並且沒有固定時間
        - 雖然可以透過 `OPTIMIZE` srcipt 手動觸發 merge 但因為會造成大量讀取與寫入，而影響效能
        - ReplacingMergeTree 適合在背景清除重複資料以節省空間，但不能保證不存在重複資料
        - 查詢時 可以加上 `FINAL` 以過濾資料

    - 語法

        {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "ReplacingMergeTree.sql">}}

    - 範例：不指定 `ver` (會刪除具有相同 `ORDER BY` 欄位的重複資料，僅保留最近 insert 的一筆)

        {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "ReplacingMergeTreeWithoutVer.sql">}}

    - 範例：指定 `ver` (允許使用 `UInt*`、`Dat`e、`DateTime` 或 `DateTime64`，僅保留最大的版本，版本相同時保留最近 insert 的一筆)

        > 這邊使用 `order_date` 做為 `ver` 範例

        {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "ReplacingMergeTreeWithVer.sql">}}

    - 實際效果

        - 新增資料

            > 分別塞入 `1882682734613504000, '2023-01-02', 100, 1, 100.001` 與 `1882682734613504000, '2023-01-01', 100, 1, 100.001`

            {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "InsertReplacingMergeTree.sql">}}

        - 不指定 `ver`

            > 因為 `2023-01-01` 較新，所以 `2023-01-02` 會被過濾掉

            ![1withoutver](https://github.com/yowko/picsbed/assets/3851540/80c56811-84af-4034-8dc2-889f86ea1409)

            ![2withoutverfinal](https://github.com/yowko/picsbed/assets/3851540/40255812-fa74-4ebe-826c-ab19f7052666)

        - 指定 `ver`：

            > 因為 `2023-01-02` 版本較高，所以 `2023-01-01` 會被過濾掉

            ![3withver](https://github.com/yowko/picsbed/assets/3851540/c94e3665-7bbb-4387-a003-1f10e288da9a)

            ![4withverfinal](https://github.com/yowko/picsbed/assets/3851540/c06397f0-65b3-465c-88db-6fa356e65e9d)

2. CollapsingMergeTree

    - 特色
        - 非同步刪除 `ORDER BY` 所指定的欄位皆相同時 `SIGN` (允許 `1`: active 與 `-1`: cancel) 成對的資料，非成對資料則保留
        - 可顯著降低儲存空間並且提高 SELECT 查詢效率
        - 查詢時 可以加上 `FINAL` 以過濾資料，但效率不佳，建議不要使用在資料量大的 table
        - 使用
    - 語法

        {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "CollapsingMergeTree.sql">}}

    - 範例

        {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "CollapsingMergeTreeSign.sql">}}

    - 實際效果

        - 新增資料

            {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "InsertCollapsingMergeTree.sql">}}

            ![5insert](https://github.com/yowko/picsbed/assets/3851540/831049f8-31c2-4527-9c37-e20071ad9331)

        - 需要匯總資料時，可以使用 `GROUP BY` 來匯總資料

            {{<gist yowko bc3dfea043713e7c914ba307a1d8ee80 "CollapsingMergeTreeGroupBy.sql">}}

            ![6sumgroupby](https://github.com/yowko/picsbed/assets/3851540/3192dfdb-2e78-4eb5-acbd-3a63bc54e211)

        - 無法使用匯總資料

            - 使用 `FINAL`

                ![6final](https://github.com/yowko/picsbed/assets/3851540/201166d2-59a2-4770-b6d2-27123d93d45f)

            - 使用 `OPTIMIZE`

                ![7optimize](https://github.com/yowko/picsbed/assets/3851540/c4c70fc8-dcec-4586-b8df-6a22ac980bac)

## 心得

- `ReplacingMergeTree`

    1. 官網沒有提到 `FINAL` 會影響，不知道實際狀況如何
    2. 不用額外增加欄位比較方便

- `CollapsingMergeTree`

    1. 可以不用有 `-1` 的 sign，但是會造成資料量變大，且效能變差
    2. 同一筆 insert 有 `1` 與 `-1` 的 sign，會保留 `1` 的資料
    3. 需要將前一筆 record 刪除時，需要先取得前一筆資料內容，這樣就多一次查詢的成本
    4. 不是每次查詢都允許匯總資料，官方不建議使用 `FINAL` 這樣就需要手動執行 `optimize table` 來清除重複資料，這樣會產生大量讀取與寫入，而影響效能

`ReplacingMergeTree` 與 `CollapsingMergeTree` 用下來，感覺完全不會想用 `CollapsingMergeTree` 限制多眉角也多，我猜測可能效能或是資源消耗較低，只是我短暫體驗下看不出差異

## 參考資料

1. [新增修改刪除查詢 ClickHouse 資料"](/clickhouse-insert-update-select-delete)
2. [Avoid Mutations](https://clickhouse.com/docs/en/optimize/avoid-mutations)
3. [ReplacingMergeTree](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replacingmergetree)
4. [CollapsingMergeTree](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/collapsingmergetree)
