---
title: "Docker Container 佔用磁碟大小"
date: 2021-02-22T09:30:00+08:00
lastmod: 2021-02-22T09:30:31+08:00
draft: false
tags: ["Docker","Container"]
slug: "docker-container-size"
---

## Docker Container 佔用磁碟大小

最近在做個功能優化的 POC，其他一個項目是評估資料儲存的大小，這個評估我打算透過 container 來進行比較 (雖然跟實際安裝 service 有一定程度的落差，但我進行的這個 POC 主要是為了比較不同 technology stack 的相對差異，不是絕對的數值)

在透過 docker 指令查詢 container 佔用 disk 大小時，對於 docker 指令所呈現的數據不太明白，查了資訊順手紀錄一下  加深印象

## 基本環境說明

1. macOS Big Sur 11.2.1
2. docker desktop 3.1.0(51484)
3. docker images

    - wurstmeister/kafka:2.13-2.7.0
    - wurstmeister/zookeeper:3.4.6

## 確認方式與說明

- 使用 `docker ps -s` 或是 `docker ps --size`

    > 官方用法說明在此 [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)

    ![1dockerpssize](https://user-images.githubusercontent.com/3851540/108661974-2ff97d80-7508-11eb-938a-9aeed80c6cfc.png)

- `SIZE` 有兩個數字

    > 這邊是根據 [Explain the SIZE column in "docker ps -s" and what "virtual" keyword means](https://github.com/docker/docker.github.io/issues/1520#issuecomment-305179362) 的個人理解，為了避免我誤導，強烈建議對照 [Explain the SIZE column in "docker ps -s" and what "virtual" keyword means](https://github.com/docker/docker.github.io/issues/1520#issuecomment-305179362)

    - 第一個數字是 container `可讀磁區` 的大小
    
        > 每個 container 都是獨立的
    
    - 第二個數字 (virtual) 是 container `可讀磁區 + 唯讀磁區` 的大小

        > 其中 `virtual` 的數字是所有使用相同 image 建立的 container 共用的 (使用相同 image 建立的所有 container 會有相同的 virtual size)

        ![2samevirtual](https://user-images.githubusercontent.com/3851540/108662202-c9c12a80-7508-11eb-9fe7-ac8e404f1812.png)

## 心得

為了避免我誤導，強烈建議對照 [Explain the SIZE column in "docker ps -s" and what "virtual" keyword means](https://github.com/docker/docker.github.io/issues/1520#issuecomment-305179362)

其中提到透過 `docker ps -s` 或是 `docker ps --size` 所取得的磁碟用量不包含下列內容

1. log file
2. 額外 mount 的 volume
3. container 啟動時從外部指定的 configuration
4. container memory 因為 swap 儲存至 disk 的部份
5. checkpoint/restore 等實驗性功能

## 參考資訊

1. [Explain the SIZE column in "docker ps -s" and what "virtual" keyword means](https://github.com/docker/docker.github.io/issues/1520#issuecomment-305179362)
2. [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
