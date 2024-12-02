---
title: "為 GCP Cloud Run 加上呼叫 key 保護"
date: 2024-09-05T00:30:00+08:00
lastmod: 2024-02-02T00:30:31+08:00
draft: false
tags: ["csharp","gcp"]
slug: "gcp-cloud-run-secure"
---

## 為 GCP Cloud Run 加上呼叫 key 保護

之前筆記  [使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護](/gcp-secure-cloud-function-with-api-key) 紀錄如何透過 Api Gateway 設定 key 來保護 Cloud Functions 避免被無差異呼叫攻擊，最近剛好有個類似的需求，雖然有著之前的經驗加上筆記 [使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護](/gcp-secure-cloud-function-with-api-key) 的幫助下，實作的速度不慢，但也不如預期的快，原因是兩個：

1. GCP 的 Serverless Solution 改版：Cloud Functions --> Cloud Run (雖然 Cloud Functions 仍然可以使用，但當然要用新的  雲端的 solution 迭代速度常常讓人措手不及)
2. 筆記 [Google Cloud Functions 發送訊息到 Google Cloud Pub/Sub](/gcp-cloud-function-pubsub) 與 [使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護](/gcp-secure-cloud-function-with-api-key) 還是有些地方不夠完整

所以趁著這次機會了解一下 Cloud Run 與補充之前筆記的不足之處

## 基本環境說明

- macOS Sonoma 14.6.1 (Apple M2 Pro)
- OrbStack 1.6.4 (17192)
- .NET SDK 8.0.401

## 設定方式

1. 建立 Cloud Run Service

    - 基本設定

        > 沒有預設值的是 `1` 與 `4`，其他部份皆可使用預設值 (`2` 是選近點的反應比較快、`3` 是個人熟悉的語言)

        ![1cloudrun](https://github.com/user-attachments/assets/b2dec88b-6566-406a-9c66-9a6e0f7c145b)

    - 程式碼

        > 以下使用預設內容做為範例

        {{<gist yowko d7c3f34e1344a2f869534d6bd61b4c76 "Function.cs">}}

    - 設定完成後紀錄端點網址，後續設定 api gateway 時會用到

        ![16runurl](https://github.com/user-attachments/assets/718df32a-3678-427c-844e-bf36c208cf46)

2. `IAM 與管理` --> `服務帳戶` --> `建立服務帳戶`

    - 服務帳戶詳細資料

        > 設定服務帳戶名稱與服務帳戶 ID

        ![2createsa](https://github.com/user-attachments/assets/0304f4ce-af8b-4daf-9362-b45c9235c1f4)

    - 將專案存取權授予這個服務帳戶 (選用)

        > 需要 `Cloud Run 叫用者` 權限

        ![3createsa](https://github.com/user-attachments/assets/1e15abe9-c9e1-4a34-81cb-aa7bdbaf1ef7))

    - 將這個服務帳戶的存取權授予使用者 (選用)

        > 這不需要額外設定，保持預設值即可

3. `API Gateway` --> `建立閘道`

    - API

        > 設定顯示名稱與 API ID

        ![4createapi](https://github.com/user-attachments/assets/532e9066-06df-4286-ad0a-69fdf7815e4f)

    - API 設定

        - 上傳 API Spec

            > - `paths./` 是 api url 後面的路徑，設定 route 的部份
            > - `paths./.post` 允許使用 `post`
            > - `paths./.post.x-google-backend.address` 的 `{cloud function url}` 要記得改成正確的值
            > - `paths./.post.parameters` 使用 url query 的方式來傳遞型態為 `string` 的 `name` 參數
            > - `paths./.post.security` 表示 `/` 這個 route 下沒有預設的 api_key
            > - `securityDefinitions` 用來指定 api_key 如何傳遞

           {{<gist yowko d7c3f34e1344a2f869534d6bd61b4c76 "test-cloudrun-api.yaml">}}

        - 設定顯示名稱並選取使用上面建立的服務帳戶

            ![5apiconfig](https://github.com/user-attachments/assets/9ed72bd8-3afb-4639-97d2-82789dcf3378)

    - 閘道詳細資料

        > 設定顯示名稱與部署位置(部置只有部份地區可以選擇)

        ![6gateway-detail](https://github.com/user-attachments/assets/30d90969-0478-4b48-b82b-36b6d0f738ab)

4. `API 與服務` --> `啟用API 與服務`

    > 要等上一步驟的 api 確定建立後才能執行啟用

    ![7apiservice](https://github.com/user-attachments/assets/c6c7efd9-b626-4768-8da2-b2d77af15c5c)

    ![8enableapi](https://github.com/user-attachments/assets/1434e3a7-8cf0-465c-af5d-d17ce6311d3b)

5. `API 與服務` --> `建立憑證` (API 金鑰)

    > 為 api 新增 crendential API Key

    ![9apikey](https://github.com/user-attachments/assets/ee078e9f-2f95-47d3-9a0a-ff498908248b)

6. `API 與服務` --> `編輯 API 金鑰`

    > 為 API Key 加上呼叫限制，限制只能 access 自訂的 api (預設可以存取所有 api)

    ![10apistrict](https://github.com/user-attachments/assets/b22516f7-e579-41f3-88fc-bf1b2ffcaeb7)

- 使用方式

    - `API Gateway` 中選取目標 API

        ![11api](https://github.com/user-attachments/assets/ef57e7bc-40c3-48fc-aecd-1bc61b78adc2)

    - `閘道` 可以取得對外的 url

        ![12gatewayurl](https://github.com/user-attachments/assets/9a326b0b-ee4b-4f7d-b909-e9ebb13a7432)

    - 實際使用

        - 未帶 api key 或是 key 不正確：會得到 `401 UNAUTHENTICATED`

            ![13unauth](https://github.com/user-attachments/assets/8423d496-03e4-42e6-ad04-b2ecd96983c9)

        - 使用正確 api key：會得到 `Hello world!`

            > api key 的使用方式是在 url 後面加上 `?key={api key}` (這個是由 step 3 中上傳的 api config 所設定的)

            ![14withkey](https://github.com/user-attachments/assets/04cf4c6c-e99f-41ef-ada7-747a3710a662))

        - 使用正確 api key 並加上 name：會得到 `Hello {name}!`

            ![15withkeyname](https://github.com/user-attachments/assets/10c7380f-4c61-470e-a4a8-fad3d4cf00bf))

## 心得

Cloud Function 與 Cloud Run 的差異

功能|Cloud Functions (第 1 代)|Cloud Functions (第 2 代) - Cloud Run
:---|:---|:---
Image registry|Container Registry 或 Artifact Registry|僅限 Artifact Registry
Request timeout|最長 9 分鐘| - HTTP 觸發，最長 60 分鐘<br/> - 事件觸發，最長 9 分鐘
規格|最多 2 個 vCPU，最高 8GB 的 RAM|最多 4 個 vCPU，最高 16GiB 的 RAM
Concurrency|每個 function instance 1 個 request|每個 function instance最多 1,000 個 request
流量分配|不支援|支援
事件類型|支援 Pub/Sub triggers、Cloud Storage triggers、Firestore triggers、Google Analytics for Firebase triggers、Firebase Realtime Database triggers、Firebase Authentication triggers、Firebase Remote Config triggers |支援 Pub/Sub triggers、Cloud Storage triggers、Firestore triggers 與 Generalized Eventarc triggers (Eventarc 所支援的任何事件類型，包括通過 Cloud Audit Logs 提供的 90 多個事件來源)
CloudEvents|只支援 Ruby、.NET 和 PHP runtime|所有語言 runtime 都支援

Cloud Run 與 Cloud Functions 就我自己使用起來體感差異不大，甚至連 dotnet 的 project template 都相同，不過後續相關的設定會些許不同，但差異也不大，原則上應該可以直接轉換

## 參考資訊

1. [Google Cloud Functions 發送訊息到 Google Cloud Pub/Sub](/gcp-cloud-function-pubsub)
2. [使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護](/gcp-secure-cloud-function-with-api-key)
3. [Cloud Run functions version comparison](https://cloud.google.com/functions/docs/concepts/version-comparison)
