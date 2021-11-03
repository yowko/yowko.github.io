---
title: "取得 Consul 中的 Key/Value"
date: 2020-04-11T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["Consul","Tools"]
slug: "consul-kv-without-encoding"
---

## 取得 Consul 中的 kv 值

最近沒做什麼專案，大部份時間都在擔任 SRE，所以常常需要確認環境設定，因此打算簡單紀錄些平常工作用到的指令，避免每次急著要用時想不起來、找不到的 script

今天就從 Consul 先來：Consul 比較有名的是服務發現功能，而我用到的是 Key/Value - 主要是用來儲存一些 json 資料。有時候系統運行不如預期時就會確認 json 資料是否正確，看看可以如何取得 Consul 中的 Key/Value 內容

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.4(43472)

    - Engine 19.03.8
    - Kubernetes v1.15.5
3. docker images

   - consul:1.7.2

        ```bash
        docker run -d -p 8500:8500 consul:1.7.2
        ```

4. Consul kv 設定

    ![1kv](https://user-images.githubusercontent.com/3851540/79047518-a1f70a00-7c49-11ea-9402-bc298b1e7d41.png)

## 問題與解決方式

- Consul kv 有提供 rest api

    - api 格式

        ```txt
        http://{ip or domain}:{port}/v1/kv/{key name}
        ```

    - api 範例

        ```txt
        http://127.0.0.1:8500/v1/kv/YowkoTest
        ```

- 預設 Value 有加密

    ```bash
    curl http://127.0.0.1:8500/v1/kv/YowkoTest
    ```

    ![2encoding](https://user-images.githubusercontent.com/3851540/79047520-a4f1fa80-7c49-11ea-884d-777e4145a112.png)

- 使用 `?raw`

    > 如果是 mac，需要在 `?` 前加上 `\`

    ```bash
    curl http://127.0.0.1:8500/v1/kv/YowkoTest?raw
    ```

    ![3raw](https://user-images.githubusercontent.com/3851540/79047522-a6232780-7c49-11ea-8ac7-60d44461bed8.png)

## 心得

超簡單的參數，只是一旦忘記，連關鍵字都不知道怎麼下 XD，自己紀錄一下不僅加深印象，日後忘記也比較好找

加上 `raw` 不僅可以取得 key 對應的 value，連帶原本的 metadata 也都一併被消去，留下純粹的 value

## 參考資訊

1. [KV Store Endpoints](https://www.consul.io/api/kv.html)
