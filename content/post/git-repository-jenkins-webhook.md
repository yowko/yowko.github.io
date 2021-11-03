---
title: "遠端 Git Repository 有變動時自動通知 Jenkins 2 進行編譯建置 (webhook)"
date: 2017-02-10T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps","Git"]
slug: "git-repository-jenkins-webhook"
aliases:
    - /2017/02/git-repository-jenkins2-webhook.html
---
## 遠端 Git Repository 有變動時自動通知 Jenkins 2 進行編譯建置 (webhook)

之前筆記 [如何使用 Jenkins 2.0 建置 .NET 專案](/jenkins-2-build-dotnet-project) 紀錄到該如何使用 Jenkins 2 來建置 .NET 專案，為了讓整個開發建置流程更加自動化也更確保品質，可以透過 Git server 所提供的 webhook 功能來 demo 該如何設定 Jenkins 2 和 Git server (將以 GitHub 為例)，讓版控 (git) 可以主動通知 build server (Jenkins 2) 發動建置

## 文章大綱

1. 設定 Jenkins 2 啟用 build trigger
2. 設定 Github 啟用 webhook
3. 測試

## 1. 設定 Jnekins 2 Build Triggers

- Trigger builds remotely (e.g., from scripts)

    ![1url](https://cloud.githubusercontent.com/assets/3851540/21797554/cb16e4fc-d74a-11e6-8669-35af659dfaa4.png)
  - 需有對外的 url (讓 GitHub 通知用)
    - 將 url 記下來，等等需要至 GitHub 設定

            ![2remourl](https://cloud.githubusercontent.com/assets/3851540/21797555/cb198e5a-d74a-11e6-96ea-c247378bf725.png)
  - 提供一組驗證用 token
    - 提供基本的安全防護

            ![3token](https://cloud.githubusercontent.com/assets/3851540/21797557/cb1b2526-d74a-11e6-80d6-7ea8fefb94c4.png)

- 針對 CSRF 調整
  - 預設啟用 CSRF 保護
  - 403 - No valid crumb was included in the request

        ![5CSrF](https://cloud.githubusercontent.com/assets/3851540/21797558/cb1f2b3a-d74a-11e6-93ca-ee49d44eb68a.png)
  - 解決方式有兩個
    - 使用 API Token(建議)
      - 1.Jenkins --> People

                ![10people](https://cloud.githubusercontent.com/assets/3851540/21882885/6f3d4bfa-d8e7-11e6-9b2d-8d8be4a590ad.png)
      - 2.People --> Configure

                ![11someone](https://cloud.githubusercontent.com/assets/3851540/21882887/6f408130-d8e7-11e6-90ea-2ec227fe97e3.png)

                ![12configure](https://cloud.githubusercontent.com/assets/3851540/21882889/6f4305e0-d8e7-11e6-8b0a-6b71d57c689a.png)
      - 3.Show API Token..

                ![13token](https://cloud.githubusercontent.com/assets/3851540/21882890/6f444e1e-d8e7-11e6-92a3-24289cadf32f.png)

                ![14token](https://cloud.githubusercontent.com/assets/3851540/21882891/6f5cb5ee-d8e7-11e6-9b12-9c0dead992c9.png)
      - 4.使用 usern 及 API TOKEN
        - `userid:token@JENKINS_URL/job/TestWebhook/build?token=TOKEN_NAME`
      - 5.proxy 環境下可 `Enable proxy compatibility`
        - 某些 HTTP 代理伺服器會將預設 Crumb 簽發程式用來計算 Nonce 值的資訊過濾掉
        - 如果還是無法解決問題，請考慮關閉 CSRF 保護

                    ![22enableproxy](https://cloud.githubusercontent.com/assets/3851540/21797730/c799bd6c-d74b-11e6-928f-cc727ac20aad.png)
    - 取消 CSRF 保護(<span style="color:red">不建議</span>)
      - 1.Manage Jenkins

                ![6manage](https://cloud.githubusercontent.com/assets/3851540/21882886/6f401aba-d8e7-11e6-9a1f-cfd951eb7fdb.png)
      - 2.Configure Global Security

                ![7security](https://cloud.githubusercontent.com/assets/3851540/21882884/6f3b055c-d8e7-11e6-8c25-bd676a1db7ad.png)
      - 3.取消 `Prevent Cross Site Request Forgery exploits`

                ![8uncsrf](https://cloud.githubusercontent.com/assets/3851540/21797562/cb3e438a-d74a-11e6-917c-fc86215fcad7.png)

                ![9uncsrf](https://cloud.githubusercontent.com/assets/3851540/21797561/cb3e5104-d74a-11e6-9eab-63957e76b4b7.png)

*如果沒有對外 domain ，可以參考 [使用 ngrok 讓本機上的網站可以被全世界看到](/ngrok)*

## 2. 以 GitHub 為例設定

- Repository Setting --> Webhooks --> Add web hook

    ![4addwebhook](https://cloud.githubusercontent.com/assets/3851540/21797556/cb1b0622-d74a-11e6-81ad-8409f4438f82.png)

- Webhook Setting

    ![15hooksetting](https://cloud.githubusercontent.com/assets/3851540/21797566/cb60dc7e-d74a-11e6-88c0-9cabf423a14d.png)
  - Payload URL

        > 填 `userid:token@JENKINS_URL/job/TestWebhook/build?token=TOKEN_NAME`
  - Content type

        > 可不用修改
  - Secret

        > 可不用修改
  - 預設使用 ssl 驗證
  - Webhook event
  - Active
- 重新發動 webhook
  - Recent Deliveries --> Redeliver

        ![16delivery](https://cloud.githubusercontent.com/assets/3851540/21797570/cb67a284-d74a-11e6-8091-acd58487dd24.png)
  - 17redeliver

        ![17redeliver](https://cloud.githubusercontent.com/assets/3851540/21797571/cb7c3a5a-d74a-11e6-80d1-5279d857a803.png)

## 3. 測試

1. 將 commit push 至 git server 上

    ![18push](https://cloud.githubusercontent.com/assets/3851540/21797572/cb85861e-d74a-11e6-94e6-4fc25263c94d.png)

2. Jenkins server 收到 git server 的 webhook

    ![19received](https://cloud.githubusercontent.com/assets/3851540/21882892/6f6108f6-d8e7-11e6-9dd8-ace82f9e20f8.png)  

3. Jenkins 開始執行 build

    ![20build](https://cloud.githubusercontent.com/assets/3851540/21882894/6f6466d6-d8e7-11e6-935c-205b9985e345.png)

    ![21success](https://cloud.githubusercontent.com/assets/3851540/21882893/6f62367c-d8e7-11e6-89a1-91fedfd94d3e.png)

## 參考資料

1. [About Webhooks](https://help.github.com/articles/about-webhooks/)
2. [GitHub Developer - Webhooks](https://developer.github.com/webhooks/)
3. [如何使用 Jenkins 2.0 建置 .NET 專案](/jenkins-2-build-dotnet-project)
4. [使用 ngrok 讓本機上的網站可以被全世界看到](/ngrok)
