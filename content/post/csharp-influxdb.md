---
title: "使用 C# 存取 InfluxDB"
date: 2019-09-15T20:40:00+08:00
lastmod: 2021-11-03T20:40:30+08:00
draft: false
tags: ["csharp","InfluxDB"]
slug: "csharp-influxdb-curd"
---

## 使用 C# 存取 InfluxDB

最近專案可能會用到 InfluxDB，之前嘗試過的經驗主要是用在監控上，不是直接使用 C# 存取，所以趁著專案開始前先試試看可以如何存取，免得延誤專案時程

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. Docker for Mac 2.1.0.2 (37877)
4. InfluxDB v1.7.8
5. NuGet package

    - InfluxData.Net 8.0.1

## 建立 InfluxDB instance

- 8086 是 HTTP API port
- ~~8083 是管理介面的 port~~ 
- ~~透過環境變數 (INFLUXDB_ADMIN_ENABLED) 來啟用管理介面~~
- 管理介面已在 InfluxDB 1.3.0 移除
- INFLUXDB_ADMIN_USER 未指定時將不會建立 管理者
- INFLUXDB_ADMIN_PASSWORD 設定管理者密碼，未指定時會直接在 standard out 顯示
- INFLUXDB_HTTP_AUTH_ENABLED 啟用認證

<del>
    ```bash
    docker run -d -p 8086:8086 -p 8083:8083 -e INFLUXDB_ADMIN_ENABLED=true influxdb
    ```
</del>

```bash
docker run -d -p 8086:8086 -e INFLUXDB_ADMIN_USER=admin -e INFLUXDB_ADMIN_PASSWORD=pass.123 -e INFLUXDB_HTTP_AUTH_ENABLED=true influxdb
```

## 使用方式

1. 建立 influxdb client

    ```cs
    var influxDbClient = new InfluxDbClient("http://localhost:8086/", "admin", "pass.123", InfluxDbVersion.Latest);
    ```

2. 建立 Database

    ```cs
    var dbName = "testDb";
    var response = await influxDbClient.Database.CreateDatabaseAsync("testDb");
    ```

3. Insert 資料

    ```cs
    var pointToWrite = new Point()
    {
        Name = "yowkoTest", // serie/measurement/table to write into
        Tags = new Dictionary<string, object>()
        {
            { "SensorId", 8 },
            { "SerialNumber", "00AF123B" }
        },
        Fields = new Dictionary<string, object>()
        {
            { "SensorState", "active" },
            { "Humidity", 431 },
            { "Temperature", 22.1 },
            { "Resistance", 34957 }
        },
        Timestamp = DateTime.UtcNow // optional (can be set to any DateTime moment)
    };

    var response = await influxDbClient.Client.WriteAsync(pointToWrite, dbName);
    ```

4. Select 資料

    ```cs
    var query = "SELECT * FROM yowkoTest WHERE SensorState = 'active' ";
    var response = await influxDbClient.Client.QueryAsync(query, dbName);
    ```

5. Update 資料

    > influxDB 沒有專屬的 update 語法，可以透過 insert 相同 Timestamp 來覆蓋資料，不過僅限於 field 內容，如是 tag 會多一個資料

    ```cs
    var now = DateTime.UtcNow;
            
    var pointToWrite = new Point()
    {
        Name = "yowkoTest", // serie/measurement/table to write into
        Tags = new Dictionary<string, object>()
        {
            { "tag1", "yowkoTag1" }
        },
        Fields = new Dictionary<string, object>()
        {
            {"purpose","forUpdate"}
        },
        Timestamp =now // optional (can be set to any DateTime moment)
    };
    
    var response = await influxDbClient.Client.WriteAsync(pointToWrite, dbName);

    var pointToWrite2 = new Point()
    {
        Name = "yowkoTest", // serie/measurement/table to write into
        Tags = new Dictionary<string, object>()
        {
            { "tag1", "yowkoTag1" }
        },
        Fields = new Dictionary<string, object>()
        {
            {"purpose","updated"}
        },
        Timestamp =now // optional (can be set to any DateTime moment)
    };
    var response2 = await influxDbClient.Client.WriteAsync(pointToWrite2, dbName);
    ```

6. Delete 資料

    ```cs
    var query = "DROP SERIES FROM yowkoTest WHERE SensorId = '8' ";
    var response = await influxDbClient.Client.QueryAsync(query, dbName);
    ```

## 心得

InfluxDB 比較偏重 insert 與 select，對於 update 與 delete 支援度不好，我想應該是設計理論與用途的關係，不過對於相關的用法、名詞、特性還是不太熟悉，過陣子較上手些後再來筆記囉

## 參考資訊

1. [influxdb](https://hub.docker.com/_/influxdb)
2. [API Reference](https://docs.influxdata.com/influxdb/v0.13/concepts/api/)
3. [InfluxDB學習筆記](https://www.itread01.com/content/1541065507.html)
