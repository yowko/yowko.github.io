---
title: "使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護"
date: 2023-04-28T00:30:00+08:00
lastmod: 2023-04-28T00:30:31+08:00
draft: false
tags: ["gcp"]
slug: "gcp-secure-cloud-function-with-api-key"
---

## 使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護

這是之前筆記 [Google Cloud Functions 發送訊息到 Google Cloud Pub/Sub](/gcp-cloud-function-pubsub) 的延伸，因為 Google Cloud Functions 不適合也沒辦法直接公開讓外部服務呼叫，所以要想個辦法來為 Google Cloud Functions 加上保護

## 基本環境說明

- 相關動作都在 GCP console 與 CLOUD SHELL 完成，沒有需要指定的環境設定值

## 設定方式

1. 建立 Service Account

    - Service account details

        > 提供 name 與 id

        ![1createsa](https://user-images.githubusercontent.com/3851540/235078559-3bd8caa6-a95e-4c0c-98db-2c4e59ef9fff.png)

    - Grant this service account access to project

        > 需要 `Cloud Functions Invoker` 權限

        ![2createsa](https://user-images.githubusercontent.com/3851540/235078570-f517c029-4132-425b-833e-212f7b2cfb7f.png)

    - Grant users access to this service account (optional)

        > 這不需要額外設定，保持預設值即可

2. Create gateway

    - API

        > 提供 name 與 id

        ![3gateway-api](https://user-images.githubusercontent.com/3851540/235078578-cedac56a-eadc-479b-baad-e1b088d823b6.png)

    - API Config

        - 上傳 API Spec

            > - `paths./publishpubsub` 是 api url 後面的路徑，設定 route 的部份
            > - `paths./publishpubsub.post` 允許使用 `post`
            > - `paths./publishpubsub.post.x-google-backend.address` 的 `{cloud function url}` 要記得改成正確的值
            > - `paths./publishpubsub.post.parameters` 使用 url query 的方式來傳遞型態為 `string` 的 `message` 參數
            > - `paths./publishpubsub.post.security` 表示 `/publishpubsub` 這個 route 下沒有預設的 api_key
            > - `securityDefinitions` 用來指定 api_key 如何傳遞


            ```yaml
            swagger: "2.0"
            info:
              title: test-function-api
              description: "Call cloud function to publish pubsub"
              version: "1.0.0"
            schemes:
              - "https"
            paths:
              "/publishpubsub":
                post:
                  summary: Call cloud function to publish pubsub
                  consumes:
                  - application/json
                  operationId: publishpubsub
                  x-google-backend:
                    address: {cloud function url}
                  parameters:
                    - name: "message"
                      in: "query"
                      description: "post message"
                      required: true
                      type: string
                  responses:
                    '200':
                      description: Successful response
                      schema:
                        type: string
                  security:
                  - api_key: []
            securityDefinitions:
              api_key:
                type: "apiKey"
                name: "key"
                in: "query"
            ```

        - 提供 name 並指定使用上面建立的 Service Account

            ![4gateway-apiconfig](https://user-images.githubusercontent.com/3851540/235078580-f257ce0d-7c3b-4fb8-b44e-502c80ffda26.png)

    - Gateway details

        > 設定 name，與部署 region，但 region 只有部份，像是 asia-east 就沒有

        ![5gateway-detail](https://user-images.githubusercontent.com/3851540/235078585-bb1548f6-8b5f-481a-b697-faedeb1f90a3.png)

3. 啟用 api

    要等上一步驟的 api 確定建立後才能執行啟用

    - `Managed Service` name 可以在 api detail 中找到

        ![6managedservice](https://user-images.githubusercontent.com/3851540/235078588-96f8eb60-b776-4797-b2e2-a313fe133e94.png)

    - 啟用後才能在 `APIs & Services` 中找到自訂的 api (我自己的經驗是要等個 5-10 分鐘才會出現)

        ![7apis](https://user-images.githubusercontent.com/3851540/235078592-4cbcf54e-cc9e-452e-beca-c3637cc2d49e.png)

    ```cloud shell
    gcloud services enable {Managed Service}
    ```

4. 為 api 新增 crendential API Key

    ![8addcredential](https://user-images.githubusercontent.com/3851540/235078600-dfa1f822-51b2-4f58-8873-b6f8233cdd4c.png)

5. 為 API Key 加上呼叫限制

    > 限制只能 access 自訂的 api (預設可以存取所有 api)

    ![9restrictapi](https://user-images.githubusercontent.com/3851540/235078608-05287a9c-458b-492b-b7c5-9daf68cd70aa.png)

6. 實際效果

    - 沒有提供 api key 會收到 401

        ![10err401](https://user-images.githubusercontent.com/3851540/235078611-15ba123d-d2b4-43e6-85cb-aec1962aef20.png)

    - 正確發送與送達

        ![11succ200](https://user-images.githubusercontent.com/3851540/235078615-8da3190f-de85-491a-841f-fc96ecb21bc1.png)

        ![12received](https://user-images.githubusercontent.com/3851540/235078619-76ab4ec8-dcd2-47c3-acb3-128833b855c7.png)

## 心得

在使用雲端服務時，我常常會感到迷惘，原因是我需要完成某個需求，而這個目的可能可以透過串接數個 service 來達成，但這幾個 service 的文件是分開的，這些分開的文件也許很詳細，但組裝起來能不能真的滿足我的需求並不明確或者是串接的設定可能有版本的落差造成感覺可以用但實際上不行

今天這筆記就是很標準的例子，我想要保護 Cloud Functions，但我不確定怎麼做才是對，或是該串接哪幾個 service 才能做到，花了好幾天才兜出個版本，還不知道是不是個好做法XD  

我覺得這應該是肇因於我不夠熟悉雲端平台的關係，只是我一直覺得花時間在熟悉雲端的投報率很差，畢竟雲端平台的迭代速度太快了，這次紀錄也遇到個活生生的例子，前一天筆記到一半，隔天發現不能用，原因是權限變了，需要額外開通XD

## 參考資訊

1. [Google Cloud Functions 發送訊息到 Google Cloud Pub/Sub](/gcp-cloud-function-pubsub)
2. [Secure Google Cloud Functions with API Gateway](https://beranger.medium.com/secure-google-cloud-functions-with-api-gateway-848f687963ae)
3. [Rate limit Google Cloud Functions with API Gateway](https://beranger.medium.com/rate-limit-google-cloud-functions-with-api-gateway-19b54bb9d9e9)
