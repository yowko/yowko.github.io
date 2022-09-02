---
title: "在 CentOS 上使用 Kafkacat (更新) : 使用 container"
date: 2022-09-02T00:30:00+08:00
lastmod: 2022-09-02T00:30:31+08:00
draft: false
tags: ["container","kafka","linux"]
slug: "kafkacat-container"
---

## 在 CentOS 上使用 Kafkacat (更新) : 使用 container

今天在新建環境進行全面測試時，其中一個環節是確認 kafka 的連線狀況，如同之前筆記 [在 CentOS 上使用 Kafkacat](/centos-kafkacat/) 提到的：不想因為需要 kafka cli 而完整下載 kafka，因此打算透過 kafkacat：[edenhill/kcat](https://github.com/edenhill/kcat) 來測試，但是 kafkacat：[edenhill/kcat](https://github.com/edenhill/kcat) 沒有提供 CentOS 的 rpm，加上目前團隊 server 還是以 CentOS 為主，過去都是透過自行編譯的方式來建立 kafkacat，只是這次編譯時發現之前筆記 [在 CentOS 上使用 Kafkacat](/centos-kafkacat/) 所紀錄的方式已失效，原本想要紀錄一下新的 build flow，但轉念一想與其重覆更新 build flow，乾脆使用 container 技術來避免各種花式 build fail 問題

## 基本環境說明

- Azure VM (Standard B2s (2 vcpu，4 GiB 記憶體)
    - OpenLogic-CentOS-7_9-gen2
- Docker version 20.10.17, build 100c701
- docker images

    - confluentinc/cp-kafkacat:7.1.1

## 使用方式

- 啟動 container 並列出 kafka 上的 metadata

    - 語法

        ```bash
        docker run --rm confluentinc/cp-kafkacat kafkacat -b {kafka_host}:{kafka_port} -L
        ```

    - 範例

        ```bash
        docker run --rm confluentinc/cp-kafkacat kafkacat -b 10.1.0.4:9092 -L
        ```

        ![1kafkacat](https://user-images.githubusercontent.com/3851540/188114317-deba8687-7bd2-4b32-9d6d-615f333bc93c.png)

- 關於 docker image 的說明可以參考 [confluentinc/cp-kafkacat](https://hub.docker.com/r/confluentinc/cp-kafkacat)

- kafkacat 的參數與使用方式請參考 kafkacat：[edenhill/kcat](https://github.com/edenhill/kcat)

## 心得

還記得之前在 CentOS 上嘗試了好幾個小時才整理出可以使用的 build flow，原本正打算開始整理新的 build，幸虧靈光乍現想到使用 container，瞬間節省了好幾個小時，果然時代在演進、科技在進步呀

## 參考資訊

1. [在 CentOS 上使用 Kafkacat](/centos-kafkacat/)
2. [edenhill/kcat](https://github.com/edenhill/kcat)
3. [confluentinc/cp-kafkacat](https://hub.docker.com/r/confluentinc/cp-kafkacat)
