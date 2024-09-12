---
title: "使用 Cloud Run Job 來處理 Error Reporting"
date: 2024-09-11T00:30:00+08:00
lastmod: 2024-09-11T00:30:31+08:00
draft: false
tags: ["csharp","gcp"]
slug: "gcp-cloud-run-job-error-reporting"
---

## 使用 Cloud Run Job 來處理 Error Reporting

原本打算透過之前筆記 [為 GCP Cloud Run 加上呼叫 key 保護](/gcp-cloud-run-secure) 的內容來處理 Error Reporting 的錯誤通知，我的實作方式是設定 Error Reporting 的 Notification 透過 webhook 的方式發送到 Cloud Run，處理結束後再透過 Error Reporting api 來將錯誤標記為已處理。

實作後發現 Error Reporting 的 Notification 會出現部份錯誤未成功發送的狀況(嘗試過 email 也有相同問題)，我沒有找到確切的原因，猜測可能是 Error Reporting 在錯誤分群時會以分鐘為單位，這樣一來就變成我在一分鐘開始處理並標記 resolved 的錯誤 group，可能在同一分鐘內又有新的錯誤事件發生，造成錯誤 group 又被標記為 open ，但卻沒有再次發送通知。如果下一分鐘又重新通知也沒問題，但這時候又遇到 Error Reporting 的另個特性：open 中的錯誤 group 不會重複發送通知。

因為這個狀況，我決定改變策略：透過 Cloud Run Job 搭配定時的 trigger 來處理 Error Reporting 的錯誤通知，今天就來紀錄一下我的實作方式。

## 基本環境說明

- macOS Sonoma 14.6.1 (Apple M2 Pro)
- .NET SDK 8.0.401
- JetBrains Rider 2024.1.6
- NuGet Library

    - Google.Cloud.ErrorReporting.V1Beta1 3.0.0-beta05

## 使用方式

1. 建立 Cloud Run Job 用的 container：使用 `CloudRunJobErrorReporting` 為 project name

    > Cloud Run Job 不像 Cloud Run Service 可以使用 online editor，所以需要自行預先建立 container image

    - Programs.cs

        {{<gist yowko ef4100426376bb536b03d8b130b66992 "Program.cs">}}

    - Dockerfile

        {{<gist yowko ef4100426376bb536b03d8b130b66992 "Dockerfile">}}

    - 上傳 container image

        - 建立 Artifact Registry： `Artifact Registry` --> `Create repository`，必填項目只有 `Name` 與 `Region`，其他預設即可

            > 也可以使用 Container Registry (舊版本)，Artifact Registry 是新版本且有更多的功能

            ![1artificat](https://github.com/user-attachments/assets/fca39d1c-7f52-437c-859e-3bf1ee13c7b7)

            ![2createrepository](https://github.com/user-attachments/assets/b82384e9-fae4-48cc-af00-13890644bd69)

        - 成功建立後取得 url (保存起來，後續 push image 會用到)

            ![3repositoryurl](https://github.com/user-attachments/assets/5f6491a7-6ed9-4671-b13a-e1d4b1b34c47)

    - build container image

        - 直接使用 docker buildx 來建立並上傳 image

            > 這適合在 gcloud sdk 已經登入或是允許登入的環境使用 (像是本機開發環境就不適合登入 prod 的資訊)

            ```shell
            docker buildx build --platform linux/amd64 ./ -t {artifact registry url}/{image name and tag} --push 
            ```

        - 建立 image 後，從 cloud shell 上傳

            - 建立 image

                ```shell
                docker buildx build --platform linux/amd64 ./ -t {artifact registry url}/{image name and tag} --load
                ```

            - 打包壓縮 image

                ```shell
                docker save {artifact registry url}/{image name and tag} |gzip > {tar name}.tar.gz 
                ```

            - 從 cloud shell 上傳 image 壓縮檔

                ![4upload](https://github.com/user-attachments/assets/c9b5c9ae-602b-4cc6-8765-dd0b74114e1d))

            - 在 cloud shell 中載入 image

                ```bash
                docker load --input {tar name}.tar.gz
                ```

            - 從 cloud shell 中 push image

                ```bash
                docker push {artifact registry url}/{image name and tag}
                ```

2. 使用上傳的 image 來建立 Cloud Run Job

    ![5createjob](https://github.com/user-attachments/assets/bafcb545-856d-4c5f-bdcb-8a1de3724158)

    ![6createjob](https://github.com/user-attachments/assets/c723398b-e9f3-4ddd-a0cf-b2022fb1cc99)

3. `IAM 與管理` --> `服務帳戶` --> `建立服務帳戶`

    - 服務帳戶詳細資料

        > 設定服務帳戶名稱與服務帳戶 ID

        ![7createsa](https://github.com/user-attachments/assets/897b2ea5-43bb-4e0c-aa6e-63afde14724e)

    - 將專案存取權授予這個服務帳戶 (選用)

        > 需要 `Cloud Run Invoker` 與 `Error Reporting User` 權限

        ![8auth](https://github.com/user-attachments/assets/4efd67d0-0a61-4990-be4d-caa844620e28)

    - 將這個服務帳戶的存取權授予使用者 (選用)

        > 這不需要額外設定，保持預設值即可

4. 設定 trigger

    > 設定 `Name`, `Frequency`, `Timezone`, `Service Account` 其他預設即可，以下使用 `*/5 * * * *` 每五分鐘執行一次為範例

    ![9triggers](https://github.com/user-attachments/assets/90a2f3f2-b297-42b4-bf2b-ba109b3ce6b5)

    ![10schedule](https://github.com/user-attachments/assets/9f100f7b-8574-4741-9e12-fa1e758ee927)

    ![11sa](https://github.com/user-attachments/assets/d9ef2bf6-5c1c-47f8-be46-ef69284149da)

## 心得

概念上與 Cloud Run Service 類似，就我個人的使用經驗來看 Cloud Run Job 可以設定 trigger 來定時執行，不過 Cloud Run Job 也少了 online editor 的功能，在建立 container image 上相對麻煩

## 參考資訊

1. [為 GCP Cloud Run 加上呼叫 key 保護](/gcp-cloud-run-secure)
2. [Running services on a schedule](https://cloud.google.com/run/docs/triggering/using-scheduler)
