---
title: "C# 在啟用 TLS 的 RabbitMQ 上收發訊息"
date: 2024-02-01T00:30:00+08:00
lastmod: 2024-02-01T00:30:31+08:00
draft: false
tags: ["docker","csharp","rabbitmq"]
slug: "csharp-tls-rabbitmq"
---

## C# 在啟用 TLS 的 RabbitMQ 上收發訊息

之前筆記 [為 RabbitMQ container 啟用 TLS 連線](/rabbitmq-tls) 提到最近 partner 為了安全性考量，在與我們介接的 RabbitMQ 上啟用 TLS 連線，連線由 port 5672 改為 port 5671，造成 application 這邊完全陣亡，也紀錄到如何為 RabbitMQ container 啟用 TLS 連線建立測試環境，今天就來紀錄一下如何使用 C# 在啟用 TLS 的 RabbitMQ 上收發訊息

application 錯誤

1. 錯誤訊息

    ```log
    Unhandled exception. RabbitMQ.Client.Exceptions.BrokerUnreachableException: None of the specified endpoints were reachable
     ---> System.IO.IOException: connection.start was never received, likely due to a network timeout
       at RabbitMQ.Client.Framing.Impl.Connection.StartAndTune()
       at RabbitMQ.Client.Framing.Impl.Connection.Open(Boolean insist)
       at RabbitMQ.Client.Framing.Impl.Connection..ctor(IConnectionFactory factory, Boolean insist, IFrameHandler frameHandler, String clientProvidedName)
       at RabbitMQ.Client.Framing.Impl.Connection..ctor(IConnectionFactory factory, Boolean insist, IFrameHandler frameHandler, ArrayPool`1 memoryPool, String clientProvidedName)
       at RabbitMQ.Client.Framing.Impl.AutorecoveringConnection.Init(IFrameHandler fh)
       at RabbitMQ.Client.Framing.Impl.AutorecoveringConnection.Init(IEndpointResolver endpoints)
       at RabbitMQ.Client.ConnectionFactory.CreateConnection(IEndpointResolver endpointResolver, String clientProvidedName)
    ```

2. 錯誤截圖

    ![4error](https://github.com/yowko/picsbed/assets/3851540/8e56df89-f02f-49d8-94df-5e139686cbc7)

## 基本環境說明

1. macOS Sonoma 14.3 (Apple M2 Pro)
2. OrbStack Version 1.3.0 (16556)
3. .NET SDK 8.0.100
4. JetBrains Rider 2023.3.3
5. Container Images

    - rabbitmq:3.12.12-management

6. NuGet Library

    - RabbitMQ.Client 6.8.1

7. rabbitmq.conf

    {{<gist yowko 70b12d23afe2b373cdcf67ebcc647f49 "rabbitmq.conf">}}

8. docker-compose.yml

    {{<gist yowko 70b12d23afe2b373cdcf67ebcc647f49 "docker-compose.yml">}}

## 設定方式

1. 建立客戶端憑證

    這個部份可以參考 [為 RabbitMQ container 啟用 TLS 連線](/rabbitmq-tls) 的說明：用同一份 ca 證書來建立 client 證書，client 證書建立方式與 server 相同，再只是需要將 client 私鑰與 client 證書匯出成 p12 檔

    - 建立 client 私鑰

        ```bash
        openssl genrsa -out client.key.pem 4096
        ```

    - 建立 CSR（Certificate Signing Request）

        ```bash
        openssl req -new -key client.key.pem -sha256 -out client.csr.pem
        ```

        - Country Name (2 letter code)：預設 `AU`
        - State or Province Name (full name) ：`Some-State`
        - Locality Name (eg, city)：預設空值
        - Organization Name (eg, company) ：預設 `Internet Widgits Pty Ltd`
        - Organizational Unit Name (eg, section)：預設空值
        - Common Name (e.g. server FQDN or YOUR name) ：預設空值
        - Email Address ：預設空值

            ![1csr](https://github.com/yowko/picsbed/assets/3851540/de68662c-e130-40d1-b78c-8e11b89883ff)

    - 使用 CA 證書和私鑰簽署 CSR 以產生 client 證書

        ```bash
        openssl x509 -req -in client.csr.pem -CA ca.cert.pem -CAkey ca.key.pem -sha256 -days 365 -out client.cert.pem
        ```

        ![2cert](https://github.com/yowko/picsbed/assets/3851540/efe36978-8a57-4d67-967b-36d0ca88f62e)

    - 將 client 私鑰與 client 證書匯出成 p12 檔

        ```bash
        openssl pkcs12 -export -out cert.p12 -in client.cert.pem -inkey client.key.pem -passin pass:pass.123 -passout pass:pass.123
        ```

    - 檢驗 client 證書有效性

        ```bash
        openssl verify -CAfile ca.cert.pem cert.p12
        ```

        ![3verify](https://github.com/yowko/picsbed/assets/3851540/34441088-51ce-4954-830f-924c4dcac29c)

2. 程式碼

    - Publish

        {{<gist yowko efbfe7c888368a4381167c981b667c80 "Publisher.cs">}}

    - Consume

        {{<gist yowko efbfe7c888368a4381167c981b667c80 "Consumer.cs">}}

## 心得

1. 原則上 publish 跟 consume 的程式跟沒有 tls 的相同，只是在連線的時候需要設定 SslOption 並指定 client 證書與對應的密碼
2. 為了避免 ssl 驗證失敗也需要在 SslOption 中加上以下設定

    ```cs
    AcceptablePolicyErrors = System.Net.Security.SslPolicyErrors.RemoteCertificateChainErrors
                                  | System.Net.Security.SslPolicyErrors.RemoteCertificateNameMismatch
                                  | System.Net.Security.SslPolicyErrors.RemoteCertificateNotAvailable
    ```

3. 另外過去我都是使用 amqpTcpEndpoints 來設定 endpoint，但有 tls 的版本我還沒有試出正確的寫法，暫時先用 uri 的方式來設定 endpoint

完整程式碼可以參考 [GitHub:yowko/rabbitmq-tls-csharp](https://github.com/yowko/RabbitMqTLSTester)

## 參考資訊

1. [TLS Support](https://www.rabbitmq.com/ssl.html)
2. [為 RabbitMQ container 啟用 TLS 連線](/rabbitmq-tls)
3. [Easy Self-Signed Certificates for ASPNET Core with TLS-Gen](https://gian-lorenzetto.medium.com/easy-self-signed-certificates-for-aspnet-core-with-tls-gen-d637a17a751a)
4. [GitHub:yowko/rabbitmq-tls-csharp](https://github.com/yowko/RabbitMqTLSTester)
