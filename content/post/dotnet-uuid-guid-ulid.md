---
title: ".NET 中的 UUID(GUID) 與 ULID"
date: 2024-07-10T00:30:00+08:00
lastmod: 2024-07-10T00:30:31+08:00
draft: false
tags: ["csharp","dotnet"]
slug: "dotnet-uuid-guid-ulid"
---

## .NET 中的 UUID(GUID) 與 ULID

先來認識一下 UUID(GUID) 與 ULID

- UUID (Universally Unique Identifier)

    > 128-bit長的唯一標識符，通常用於生成唯一ID

    - UUID Version 1：基於時間和 MAC address
        - 結構：60 bit 代表時間戳，48 bit 代表節點（通常是 MAC address），14 bit 代表時間序列
        - 特點：基於時間和 MAC address 生成，保證 UUID 的唯一性。
        - 用途：適用於需要基於時間順序的唯一標識，如生成分佈式系統中的ID。
        - 缺點：可能會暴露生成 UUID 的機器的 MAC address 、在高並發系統中，依賴於時鐘和序列號生成，可能會導致碰撞
    - UUID Version 2：DCE 安全
        - 結構：類似於版本1，但加入了區域識別符，並使用 POSIX UID/GID。
        - 特點：為 DCE（Distributed Computing Environment）設計，用於某些特定安全應用場景。
        - 用途：較少見，主要用於需要安全性加強的應用場景
        - 缺點：支持的 library 和工具較少、依賴於特定的操作系統和安全上下文，使用上有一定的局限性
    - UUID Version 3：基於名稱的 MD5 雜湊
        - 結構：使用MD5雜湊函數，將名稱空間和名稱雜湊為 128 bit 長的 UUID。
        - 特點：對於同一名稱空間和名稱的輸入，總是會生成相同的 UUID。
        - 用途：適用於需要生成基於特定名稱的唯一標識，如生成基於 URL 的唯一 ID。
        - 缺點：使用 MD5 雜湊算法，存在碰撞風險且安全性較低、不適用於需要完全隨機的唯一標識的情況
    - UUID Version 4：隨機生成 (.NET 的 GUID 即實作為 UUID Version 4)
        - 結構：122 bit 是隨機生成的，其餘 6 bit 設定為固定值以標識 UUID version 4。
        - 特點：完全隨機生成，不依賴於任何機器信息或時間戳。
        - 用途：適用於大多數需要唯一標識的情況，如 API 密鑰、交易 ID 等。
        - 缺點：不適合需要按時間順序排序的應用場景、由於索引效率低下，可能會導致資料庫出現性能瓶頸
    - UUID Version 5：基於名稱的 SHA-1 雜湊
        - 結構：類似於版本3，但使用 SHA-1 雜湊函數。
        - 特點：對於同一名稱空間和名稱的輸入，總是會生成相同的 UUID，並且相比版本3具有更高的雜湊強度。
        - 用途：適用於需要更高安全性的基於名稱的唯一標識。
        - 缺點：使用 SHA-1 雜湊算法，仍然存在一定的碰撞風險和安全問題（SHA-1被認為不再安全、不適用於需要完全隨機的唯一標識的情況
    - UUID Version 6、7、8 ：都是新的提案，目前尚未被廣泛使用，主要目的是改進基於時間的UUID生成方法
    - UUID Version 7：(.NET 9 預計支援 UUID Version 7)

        > 討論可以看 [Extend System.Guid with a new creation API for v7](https://github.com/dotnet/runtime/issues/103658?WT.mc_id=DOP-MVP-5002594)

        - 結構：
            - 時間戳（48 bit）：表示自 Unix 紀元（1970年1月1日）以來的毫秒數，精確到毫秒。
            - 版本號（4 bit）：固定為 7，表示這是 UUID Version 7。
            - 變體位（2 bit）：用於標識 UUID 的變體，通常固定為標準的 UUID 變體。
            - 隨機數（74 bit）：由隨機數生成器產生，保證唯一性。
        - 特點：時間排序、高唯一性、無機器信息、不需要複雜的配置或依賴於特定硬件
        - 用途：適合分佈式系統與數據庫 primary key、需要按時間排序的應用場景
        - 缺點：如果時間不同步可能會導致問題、比版本 4 的純隨機數稍大，需要處理時間戳部分

- ULID (Universally Unique Lexicographically Sortable Identifier)

    > 改進的唯一標識符，為了解決 UUID（特別是 version 4）的缺點：不支持按時間排序

    - 結構：
        - 48 bit 時間戳：表示自 Unix 紀元（1970年1月1日）以來的毫秒數，可以精確到毫秒
        - 80 bit 隨機數：使用隨機數生成器生成的隨機數
    - 特點：對於同一名稱空間和名稱的輸入，總是會生成相同的 UUID，並且相比版本3具有更高的雜湊強度。
    - 用途：特別適合需要按時間排序且對唯一性要求高的應用場景
    - 缺點：同一毫秒內生成大量 ULID，可能會導致碰撞、隨機數生成器的質量不高，可能會增加碰撞風險、在極高並發的情況下，特別是單個節點在同一毫秒內生成大量 ULID，可能會導致生成速度瓶頸

過去在系統中常使用 GUID 來當作唯一識別碼，不過因為 GUID 的隨機性讓 db 的 index 效率低下，在大型系統中會受到不小的限制，後來我們後來改用 snowflake 剛好前段時間看到 [UUID and ULID in .NET: Maximizing Efficiency in Unique Identifiers](https://medium.com/codenx/uuid-and-ulid-in-net-maximizing-efficiency-in-unique-identifiers-a94c41177128) 分享 UUID 與 ULID，原本才想要試試 ULID 而已，又看到 .NET 9 預計支援 UUID Version 7 ([Extend System.Guid with a new creation API for v7](https://github.com/dotnet/runtime/issues/103658?WT.mc_id=DOP-MVP-5002594))，所以一併整理一下 GUID、ULID 與 UUID Version 7 的特性與優缺點

## 基本環境說明

- macOS Sonoma 14.5 (Apple M2 Pro)
- OrbStack 1.6.3 (17138)
- .NET SDK 8.0.101
- .NET SDK 9.0.0-preview.7
- JetBrains Rider 2024.1.4
- NuGet packages
    - Ulid 1.3.3

## 實際使用

1. GUID (UUIDv4)

    ```cs
    Guid newGuid = Guid.NewGuid();
    ```

2. ULID

    - 安裝 NuGet package

        ```bash
        dotnet add package Ulid
        ```

    - 產生 ULID

        ```cs
        Ulid newUlid = Ulid.NewUlid();
        ```

3. GUID (UUIDv7)

    > .NET SDK 9.0.0-preview.7 才支援

    ```cs
    Guid newGuid = Guid.CreateVersion7();
    ```

## 心得

單就規格來看，UUIDv7 與 ULID 差距不大，甚至 ULID 的隨機位更大，理論上更不易出現碰撞，不過因為 UUIDv7 是 .NET 官方支援，所以在使用上會更方便，只是目前還是 preview 版本，等待 .NET 9 正式版釋出後，再來比較實際的效能差異

## 參考資訊

1. [Extend System.Guid with a new creation API for v7](https://github.com/dotnet/runtime/issues/103658?WT.mc_id=DOP-MVP-5002594)
2. [UUID v7 in .NET 9](https://steven-giesel.com/blogPost/ea42a518-4d8b-4e08-8f73-e542bdd3b983)
3. [UUID and ULID in .NET: Maximizing Efficiency in Unique Identifiers](https://medium.com/codenx/uuid-and-ulid-in-net-maximizing-efficiency-in-unique-identifiers-a94c41177128)
