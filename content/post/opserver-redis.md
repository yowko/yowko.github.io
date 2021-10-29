---
title: "如何使用 Opserver 來監控 Redis"
date: 2017-03-16T02:43:34+08:00
lastmod: 2021-10-29T00:42:34+08:00
draft: false
tags: ["Tools","Redis"]
slug: "opserver-redis"
aliases:
    - /2017/03/opserver-redis.html
---
## 如何使用 Opserver 來監控 Redis

之前參加 twmvc 活動，由阿砮介紹 Stack Overflow open source 的監控專案 - OpServer，不僅是 open source 專案又是 .net MVC 寫的，實在非常有親切感，之前因為專案需要曾經嘗試安裝，最近為團隊安裝時發現又忘得差不多了XD 所以筆記一下

## 文章大綱

1. 安裝 Opserver
2. Opserver 安全性設定
3. 監控 Redis

## 安裝 Opserver

1. clone
    - 從 [GitHub](https://github.com/opserver/Opserver) 下載(clone)
2. 解壓縮檔案
3. 開啟專案並設定啟始專案
    - 預設為 `Opserver.Core`

        ![1error](https://cloud.githubusercontent.com/assets/3851540/21705712/88ede1d0-d3fc-11e6-837c-d05d9d84f8cb.png)

    - `Opserver` 專案，按右鍵，選 `Set as StartUp Project`

        ![2startup](https://cloud.githubusercontent.com/assets/3851540/21705710/88ed648a-d3fc-11e6-9034-5ad2b8e8c919.png)

4. 編譯 (Build)

## Opserver 安全性設定

- 沒找到 `SecuritySettings.config` 的錯誤

    ![3needconfig](https://cloud.githubusercontent.com/assets/3851540/21705708/88ab5a7c-d3fc-11e6-8b10-761fff9a2b89.png)
- 加入 `SecuritySettings.config`
  - `Opserver` 專案, `Config` 資料夾下有 `SecuritySettings.config.example`

        ![4securityconfig](https://cloud.githubusercontent.com/assets/3851540/21705714/88f073f0-d3fc-11e6-8d50-5c2448184ad6.png)
  - rename `SecuritySettings.config.example` 為 `SecuritySettings.config`
  - 依需求設定權限(在預設網址後面加上 `/about` 可以顯示現行安全設定)
    - AD (default)

            >`<SecuritySettings provider="AD" />`

            ![5AD](https://cloud.githubusercontent.com/assets/3851540/21705716/89100170-d3fc-11e6-9dc7-8b3576ba93b4.png)
    - alladmin

            >`<SecuritySettings provider="alladmin" />`

            ![6alladmin](https://cloud.githubusercontent.com/assets/3851540/21705715/88f40ab0-d3fc-11e6-96bf-c8d44582fac3.png)
    - View All
            >`<SecuritySettings provider="" />`

            ![7viewall](https://cloud.githubusercontent.com/assets/3851540/21705717/89103334-d3fc-11e6-8a9b-b024b7c8e224.png)
  - 如果不是使用 `AD` ，畫面需要帳號密碼，請使用 `admin`/`admin`

        ![login](https://cloud.githubusercontent.com/assets/3851540/26041694/7a9ec8c2-3961-11e7-881c-b62d45245d6b.png)

## 監控 Redis

![8nomonitor](https://cloud.githubusercontent.com/assets/3851540/21705718/89124a5c-d3fc-11e6-96b7-0e12d6e4ecd5.png)

- 加入 `RedisSettings.json`

        ![9redisconfig](https://cloud.githubusercontent.com/assets/3851540/21705709/88ccacfe-d3fc-11e6-877a-c12845224bb6.png)
  - rename `RedisSettings.json.example` 為 `RedisSettings.json`
- `RedisSettings.json` 連線資訊有兩種設定方式
    1. 只設定 `Servers`
        - 外層 `name` 為 ip or host name
        - 內層 `name` 為顯示名稱; port 為 redis port 號

        ```json
        {
          "Servers": [{
                  "name": "127.0.0.1",
                  "instances": [ { "name": "local", "port": "6379" } ]
                }
            ]
        }
        ```

    2. `allServers` 與 `Servers` 搭配
        - `allServers` instances name 表示顯示名稱;port 為 redis port 號
        - `Servers` name 填 host ip or name

        ```json
        {
          "allServers": {
            "name": "test",
            "instances": [
              {
                "name": "local",
                "port": "6379"
              }
            ]
          }
          ,
          "Servers": [{ "name": "127.0.0.1" } ]
        }
        ```

- connection 邏輯可以參考 `\Opserver.Core\Data\Redis\RedisModule.cs`

- 預設值可以參考 `\Opserver.Core\Settings\RedisSettings.cs`
  - 更新頻率(RefreshIntervalSeconds) 預設 `30 秒`
  - redis instance 預設 port `6379`

## 20170324 補充：需要密碼連線 (requirepass)

如果 redis 需使用密碼才能連線，請參加以下設定方式

1. 只設定 `Servers`
    - 在 `Servers` 的 `instances` object 中加上 `password` 設定

        ```json
        "Servers": [
            {
              "name": "127.0.0.1",
              "instances": [ { "name": "local", "port": "6379","password":"password" } ]
            }
        ]
        ```

2. `allServers` 與 `Servers` 搭配
    - 在 `allServers` 的 `instances` object 中加上 `password` 設定

        ```json
        "allServers": {
            "name": "test",
            "instances": [
                {
                    "name": "local",
                    "port": "6379",
                    "password":"password"
                }
            ]
        },
        "Servers": [{ "name": "127.0.0.1" } ]
        ```

## 監控結果

1. 列表頁

    ![10list](https://cloud.githubusercontent.com/assets/3851540/21705711/88ed353c-d3fc-11e6-9975-6281be4901f8.png)
2. 詳細頁

    ![11detail](https://cloud.githubusercontent.com/assets/3851540/21705713/88edd35c-d3fc-11e6-8af8-e71e7e01294a.png)
  
    ![12detail](https://cloud.githubusercontent.com/assets/3851540/21837148/b71e6a5a-d804-11e6-86b5-a3038184fc81.png)

## 心得

設定的工不多，只是文件稍嫌不足，屬性名稱也多有重複，不是很好理解，但整體架設還是很方便

## 參考資料

1. [Opserver GitHub](https://github.com/opserver/Opserver)
2. [OpServer 監控服務的解決方案簡報](https://dotblogs.com.tw/wuanunet/2015/11/27/introduce_stackoverflow_opserver_for_twmvc19)
