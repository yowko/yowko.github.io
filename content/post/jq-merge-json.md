---
title: "使用 jq 達成覆寫 json key 相同的效果"
date: 2021-11-16T00:30:00+08:00
lastmod: 2021-11-16T00:30:31+08:00
draft: false
tags: ["Tools","jq"]
slug: "jq-merge-json"
---

## 使用 jq 達成覆寫 json key 相同的效果

過去網站開發需要的 config 都是透過 [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#appsettingsjson) 來處理，雖然沒有用過其他工具，不過使用上沒遇到什麼問題，也就一直延用至今

過去為了避免修改 config 還需要重新 build image，已經將 config 獨立出來統一管理，存放在 consul 的 kv (key/value) 中，但背後的 config 內容還是透過 .NET 程式來處理，最近在進行架構優化，打算將處理 config 的工作從 .NET application 移除

首先我以為會遇到最大的問題是 json config 在不同環境時需要進行覆寫部份 config key

- appsettings.json

    ```json
    {
        "redisconnection": "192.168.80.3:7000,192.168.80.3:7001,192.168.80.3:7002,192.168.80.3:7003,192.168.80.3:7004,192.168.80.3:7005, password=pass.123",
        "EnableHttps": true 
    }
    ```

- appsettings.Development.json

    ```json
    {
        "redisconnection": "127.0.0.1:6379, password=pass.123"
    }
    ```

- 依環境決定 config 內容

    - prod

        > 使用預設的 `appsettings.json`

    - Development

        > 透過環境變數來決定要額外套用的 config，只有 `redisconnection` ，所以只覆寫 `redisconnection`

        ```json
        {
            "redisconnection": "127.0.0.1:6379, password=pass.123",
            "EnableHttps": true 
        }
        ```

經過短暫的 google，發現 `jq` 就已經完全支援這樣的操作，而且使用上更為簡單便利，讓我一度懷疑 .NET 的做法是不是致敬 `jq`，今天就來紀錄一下如何使用 `jq` 達成覆寫 json key 相同的效果

## 基本環境說明

1. macOS Big Sur 11.6
2. jq-1.6

## 使用方式

- 語法

    ```bash
    jq -s '.[0] * .[1]' {default json file} {json file by env}
    ```

- 範例

    ```bash
    jq -s '.[0] * .[1]' appsettings.json appsettings.Development.json 
    ```

    ![1output](https://user-images.githubusercontent.com/3851540/141885762-b3a0459d-4aa5-4cf4-9562-c1643180cc49.png)

- 參數說明

    > 以下是我個人的理解，有錯要麻煩大大指正

    - `-s`

        > 將 json 檔案內容一次讀出，而不是逐一處理

    - `.[0]`

        > 第一份 json 檔案內容

    - `*`

        > 遞迴對 json 做 merge

    - `.[1]`

        > 第二份 json 檔案內容

## 心得

語法使用上很簡單，不過對於語法的真諦就我目前的理解能力跟經驗還無法充份掌握，說白的就是能兜出可用的版本，無法靈活應用，簡單紀錄一下圖個經驗累積，希望日後慢慢可以上手了

## 參考資訊

1. [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#appsettingsjson)
2. [jq](https://stedolan.github.io/jq/)
3. [jqplay](https://jqplay.org/)
