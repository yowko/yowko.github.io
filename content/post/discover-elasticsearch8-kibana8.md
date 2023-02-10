---
title: "如何在 Kibana 8 與 Elasticsearch 8 上看到資料"
date: 2023-02-09T00:30:00+08:00
lastmod: 2023-02-09T00:30:31+08:00
draft: false
tags: ["efk","elasticsearch","kibana"]
slug: "discover-elasticsearch8-kibana8"
---

## 如何在 Kibana 8 與 Elasticsearch 8 上看到資料

最近打算將 redis slowlog 倒進 Elasticsearch 中 (詳細資料可以參考 [使用 filebeat 將 Redis slowlog 存至 Elasticsearch](/redis-slowlog-elasticsearch))，讓開發團隊可以更快速方便的查到相關資訊，開發過程偶有波折在所難免，但沒想到竟然連驗證結果是否符合預期都卡到不行：我想透過 filebeat 將 redis slowlog 打進 Elasticsearch，不過我在 Kibana 上卻一直找不到如何建立 index template，當然也看不到資料，後來跟同事研究後總算找到設定方式，為了加深印象快速紀錄一下

以往熟悉的按下 `Discover` 會引導建立 index template，現在改為安裝 `Integrations`，152 個項目卻找不到想要的 filebeat

![1discover](https://user-images.githubusercontent.com/3851540/218040425-cfc92aba-0344-4335-aeba-973d8c42cd74.png)

![2addintegrations](https://user-images.githubusercontent.com/3851540/218040430-a42c6065-436f-4626-9274-27b59142398f.png)

![3integrations](https://user-images.githubusercontent.com/3851540/218040433-09fb566a-af9e-476c-a1f8-4d06e9e5cc45.png)

## 基本環境說明

1. macOS Ventura 13.2
2. docker desktop 4.2.0(70708)
    - Engine 20.10.10
    - Compose 1.29.2
3. docker images
    - elastic/elasticsearch:8.6.1
    - elastic/kibana:8.6.1

## 設定方式

1. 點選左邊選單的 `Management`

    ![4management](https://user-images.githubusercontent.com/3851540/218040437-6a0d06ac-9a39-455c-885d-a905eb6b0187.png)

2. 檢查 index template 是否已建立： `Management` --> `Data` --> `Index Templates`

    ![5indextemplates](https://user-images.githubusercontent.com/3851540/218040441-37f78c34-87b6-42e7-86e7-ded00464cd4a.png)

3. 使用 index template 建立 Kibana data view

    ![6createdataview](https://user-images.githubusercontent.com/3851540/218040442-bc22ce61-8beb-41fc-8da6-410a307028f6.png)

    ![7savedataview](https://user-images.githubusercontent.com/3851540/218040444-039de2b3-487d-40da-a37d-27f5e86f6bbf.png)

4. 在回到 `Analytics` --> `Discover` 即可看到資料

    ![8discoverwithresults](https://user-images.githubusercontent.com/3851540/218040446-653de1ec-17a4-44de-88af-29e118617418.png)

## 心得

找到設定方式後，我想了一下為什麼第一時間不會設定的原因：

1. `Analytics` --> `Discover` 不像過去引導建立 index 而是接 `Add Integrations` 其中又沒有 filebeat 的選項
2. `Index Templates` 與 `Kibana data view` 不是首頁左邊選單直接顯示的子項目

## 參考資訊

1. [使用 filebeat 將 Redis slowlog 存至 Elasticsearch](/redis-slowlog-elasticsearch)
