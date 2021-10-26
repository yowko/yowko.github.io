---
title: "取得 kafka 運行中的設定值"
date: 2021-10-26T00:39:29+08:00
lastmod: 2021-10-26T00:39:29+08:00
draft: false
tags: ["kafka"]
slug: "kafka-current-config"
---

## 取得 kafka 運行中的設定值

kafka 的 config 預設儲存在 `kafka 根目錄/config/server.properties` ，一般情境下可以直接查看該檔案來取得設定值，但有些情況會讓這個檔案失去提供正確資訊的功用，以下舉兩個我自己遇到的狀況

1. `server.properties` 未設定(沒有主動覆蓋 default 設定)，但不清楚 default 設定是什麼
2. `server.properties` 已修改，但因為暫時無法中斷服務而未重啟 kafak 進行套用 (我自己會先修改，downtime 時只需重啟，避免當下手忙腳亂)

時間一久或是其他同事調整了還沒重啟，這時就需要查查當前 kafka instance 所使用的設定值，之前也用過幾次，畢竟不常用常常執行完就忘了語法，每次需要時就得重新再找，還是乾脆自己筆記一下，至少知道哪邊找

## 基本環境設定

1. CentOS 7.9.2009 (Azure cognosys centos-7-9-free)
2. Scala 2.13
3. kafka 2.8.1

## 取得方式

- 語法

    ```bash
    /home/kafka/bin/kafka-configs.sh --bootstrap-server {kafka server}:{kafka port} --all --describe --entity-type brokers --entity-name {broker id}
    ```

- 範例

    ```bash
    /home/kafka/bin/kafka-configs.sh --bootstrap-server 127.0.0.1:9092 --all --describe --entity-type brokers --entity-name 0
    ```

## 心得

- 實際效果

    ![currentconfig](https://user-images.githubusercontent.com/3851540/138682804-cc98a968-959b-4325-a478-8c0362f4cb5c.png)

看得出來資料很多很雜，但至少資訊是完整的

- 圖以同時檢視現在設定值與預設值

    ![2configdefault](https://user-images.githubusercontent.com/3851540/138806753-688b7878-81c6-4575-9506-e30d2feaa408.png)

## 參考資訊

1. [get current config in kafka](https://stackoverflow.com/questions/49406618/get-current-config-in-kafka)
