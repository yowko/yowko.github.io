---
title: "在 CentOS 上使用 Kafkacat"
date: 2019-12-01T10:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["CentOS","Kafka"]
slug: "centos-kafkacat"
---

## 在 CentOS 上使用 Kafkacat

之前筆記 [在CentOS 上安裝 Apache Kafka cluster](/kafka-cluster-centos/) 與 [在 CentOS 上安裝 Apache Kafka cluster - One Script](/kafka-cluster-centos-script/) 都紀錄到如何在 CentOS 上安裝 Apache Kafka，不過順利安裝與成功安裝還是存在差異的，過去我都是直接透過 Kafka 內建的 cli 工具來進行驗證不需要額外下載安裝其他工具，只是最近搭配 Kubernetes 使用時常會有需要確認連線的情境，這時候再下載完整的 Kafka 似乎顯得多餘，今天就來紀錄一下如何在 CentOS 上使用 Kafkacat

Kafkacat 是由 c 語言開發的 Kafka client，不需要 Java，非常輕巧，在 debian、Ubuntu 可以使用 `apt-get install kafkacat` 進行安裝，而 macOS 上可以透過 `brew install kafkacat` 安裝，而對於 centOS 就沒那麼 friendly 了，這也是今天會紀錄的原因

## 基本環境說明

1. Linux (centos 7.7.1908)
2. librdkafka 0.11.4-1.el7

## 安裝 kafkacat

1. 安裝 gcc-c++,git 與 librdkafka

    ```bash
    yum update -y && yum install gcc-c++ git librdkafka-devel -y
    ```

2. 下載 kafkacat

    ```bash
    git clone https://github.com/edenhill/kafkacat
    ```

3. 設定 config

    ```bash
    cd kafkacat && ././configure
    ```

4. 編譯並安裝

    ```bash
    make && make install
    ```

## 實際使用

1. producer

    > 以下語法會讓 producer 持續監聽 stdin 來傳送 message

    ```bash
    kafkacat -P -b node3:9092 -t test
    ```

2. consumer

    > 以下語法會讓 consumer 持續監聽並透過 stdout 來輸出收到的 message

    ```bash
    kafkacat -b node3:9092 -t test
    ```

## 心得

不知道為什麼 kafkacat 沒有加入 yum 中，我一度想要在 CentOS 中安裝 apt-get，只是找了些資料後發現相關安裝方法似乎都過期不適用了(我不是十分確定，有錯請指教)，於是最後選擇透過編譯原始碼的方式來處理，只是這麼一來(額外安裝了編譯器)是否有比下載 kafka (想要使用內建的 cli)來得輕巧就說不準了，但還是紀錄一下方便自己查閱，也讓網路上的諸方高手批評指教

## 參考資訊

1. [edenhill/kafkacat](https://github.com/edenhill/kafkacat)
2. [[빅데이터 인프라] Kafkacat 을 centos7에서 설치하기 / kafkacat install on centos7](https://league-cat.tistory.com/292)
