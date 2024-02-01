---
title: "為 RabbitMQ container 啟用 TLS 連線"
date: 2024-01-30T00:30:00+08:00
lastmod: 2024-01-30T00:30:31+08:00
draft: false
tags: ["docker","csharp","rabbitmq"]
slug: "rabbitmq-tls"
---

## 為 RabbitMQ container 啟用 TLS 連線

最近 partner 為了安全性考量，在與我們介接的 RabbitMQ 上啟用 TLS 連線，連線由 port 5672 改為 port 5671，原以為是簡單的調整，想不到 application 這邊完全陣亡，為了釐清問題，首先第一步就是還原環境，因此就來紀錄一下如何為 RabbitMQ container 啟用 TLS 連線

## 基本環境說明

1. macOS Sonoma 14.3 (Apple M2 Pro)
2. OrbStack Version 1.3.0 (16556)
3. OpenSSL 3.1.3 19 Sep 2023 (Library: OpenSSL 3.1.3 19 Sep 2023)
4. Container Images

    - rabbitmq:3.12.12-management

## 設定方式

1. 建立 TLS 證書

    - 建立 CA 私鑰

        ```bash
        openssl genrsa -out ca.key.pem 4096
        ```

    - 建立 CA 證書

        ```bash
        openssl req -x509 -new -key ca.key.pem -subj "/C=TW/ST=TAIWAN/O=yowko" -sha256 -days 365 -out ca.cert.pem
        ```

    - 建立 server 私鑰

        ```bash
        openssl genrsa -out server.key.pem 4096
        ```

    - 建立 CSR（Certificate Signing Request）

        ```bash
        openssl req -new -key server.key.pem -sha256 -out server.csr.pem
        ```

        - Country Name (2 letter code)：預設 `AU`
        - State or Province Name (full name) ：`Some-State`
        - Locality Name (eg, city)：預設空值
        - Organization Name (eg, company) ：預設 `Internet Widgits Pty Ltd`
        - Organizational Unit Name (eg, section)：預設空值
        - Common Name (e.g. server FQDN or YOUR name) ：預設空值
        - Email Address ：預設空值

            ![1csr](https://github.com/yowko/picsbed/assets/3851540/b5902133-6b3b-421f-a095-56c5475e6a0f)

    - 使用 CA 證書和私鑰簽署 CSR 以產生 server 證書

        ```bash
        openssl x509 -req -in server.csr.pem -CA ca.cert.pem -CAkey ca.key.pem -sha256 -days 365 -out server.cert.pem
        ```

        ![2cert](https://github.com/yowko/picsbed/assets/3851540/8cd043e5-ce57-4a90-9b6c-6cd48b5f3063)

    - 檢驗 server 證書有效性

        ```bash
        openssl verify -CAfile ca.cert.pem server.cert.pem
        ```

        ![3verify](https://github.com/yowko/picsbed/assets/3851540/141ac964-6175-42c0-a2e6-f70b172fdfd2)

2. RabbitMQ 設定

    - rabbitmq.conf

        {{<gist yowko 70b12d23afe2b373cdcf67ebcc647f49 "rabbitmq.conf">}}

    - docker-compose.yml

        {{<gist yowko 70b12d23afe2b373cdcf67ebcc647f49 "docker-compose.yml">}}

## 心得

現在回過頭看，流程並不複雜，但過程中因為我個人想要直接用指令完成所有動作，略過 prompt 的互動操作，想不到這個念頭讓我浪費不少時間，原因有下面幾個：

1. CSR 加上 subject 後不會跳密碼輸入 (個人還是想要有密碼保護)
2. CSR 加上 subject 驗證會 fail (這個不知道原因，結論就是不能用)
3. 略過 CSR 直接建立 server 證書，但仍是驗證 fail

    ```bash
    openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
    ```

    ![4fail](https://github.com/yowko/picsbed/assets/3851540/104aea81-c20f-4df2-b4f7-8399e86a30db)

相較於 openssl，rabbitmq 設定就簡單多了，只要在 `rabbitmq.conf`，需要留意的是 rabbitmq config 有新舊版本的差異，新版本在視覺上簡約許多，以下是相同設定參考比較看看：

1. 新版本 (副檔名：`.conf`)

    {{<gist yowko 70b12d23afe2b373cdcf67ebcc647f49 "rabbitmq.conf">}}

2. 舊版本 (副檔名：`.config`)

    {{<gist yowko 70b12d23afe2b373cdcf67ebcc647f49 "rabbitmq.config">}}

## 參考資訊

1. [TLS Support](https://www.rabbitmq.com/ssl.html)
2. [OpenSSL Working with SSL Certificates, Private Keys, CSRs and Truststores](https://gist.github.com/mohanpedala/468cf9cef473a8d7610320cff730cdd1#generate-a-self-signed-certificate-from-an-existing-private-key-and-csr)
3. [Configuration](https://www.rabbitmq.com/configure.html#config-file)
