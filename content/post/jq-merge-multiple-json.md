---
title: "jq 合併多個 json"
date: 2023-07-31T00:30:00+08:00
lastmod: 2023-07-31T00:30:31+08:00
draft: false
tags: ["json","jq"]
slug: "jq-merge-multiple-json"
---

## jq 合併多個 json

之前筆記 [使用 jq 達成覆寫相同 json key 的效果](jq-merge-json/) 紀錄到如何使用 jq 處理 json 讓 config 可以在不同環境有不同設定值，使用上沒遇到問題，只是最近想要強化部份資安流程，所以想要將敏感資訊抽出來管理，處理方式就會需要多 merge 一次：`appsettings.json` + `appsettings.Prod.json` + `sre.Prod.json`

- 原始 json

    1. appsettings.json

        ```json
        {
            "redisconnection": "localhost:6379, password=pass.123",
            "EnableHttps": false,
            "Env": "Default"
        }
        ```

    2. appsettings.Prod.json

        ```json
        {
            "EnableHttps": true,
            "Env": "prod"
        }
        ```

    3. sre.Prod.json

        ```json
        {
            "redisconnection": "192.168.80.3:7000,192.168.80.3:7001,192.168.80.3:7002,192.168.80.3:7003,192.168.80.3:7004,192.168.80.3:7005, password=Pa$$w0rd"
        }
        ```

- 預期輸出

    ```json
    {
        "redisconnection": "192.168.80.3:7000,192.168.80.3:7001,192.168.80.3:7002,192.168.80.3:7003,192.168.80.3:7004,192.168.80.3:7005, password=Pa$$w0rd",
        "EnableHttps": true,
        "Env": "prod"
    }
    ```

## 基本環境說明

1. macOS Ventura 13.4
2. jq-1.6

## 使用方式

1. 語法 1： `jq -s ".[0] * .[1] * .[2]" file1.json file2.json file3.json`

    > 這是之前筆記 [使用 jq 達成覆寫相同 json key 的效果](jq-merge-json/) 的延伸，概念就是直接加上第三個檔案

    ```bash
    jq -s ".[0] * .[1] * .[2]"  appsettings.json appsettings.Prod.json sre.Prod.json
    ```

2. 語法 2： `jq -s add file1.json file2.json file3.json`

    > 這是從 [How to merge multiple json files in a directory with jq or any tool?](https://stackoverflow.com/questions/63046989/how-to-merge-multiple-json-files-in-a-directory-with-jq-or-any-tool) 看到的語法，簡單到覺得不可能 work XD

    ```bash
    jq -s add appsettings.json appsettings.Prod.json sre.Prod.json
    ```

![1result](https://github.com/yowko/picsbed/assets/3851540/250c8081-73c2-48a9-9155-10778566aac3)

## 心得

經過上次 [使用 jq 達成覆寫相同 json key 的效果](jq-merge-json/) 的經驗，我以為我已經對 jq 有初步認識，但小小的改變還是讓我苦手不已 @@"

當然個人功力不足是主因，但我覺得 [./jq](https://jqlang.github.io/jq/) 官方文件不是很完善也是問題，原本想要嘗試理解 `語法2` 的概念，但卻沒找到相關說明，或是我誤會了文件的內容，可能悟性還有待加強

## 參考資訊

1. [使用 jq 達成覆寫相同 json key 的效果](jq-merge-json/)
2. [How to merge multiple json files in a directory with jq or any tool?](https://stackoverflow.com/questions/63046989/how-to-merge-multiple-json-files-in-a-directory-with-jq-or-any-tool)
3. [./jq](https://jqlang.github.io/jq/)
