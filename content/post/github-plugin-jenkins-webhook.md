---
title: "GitHub Repository 有變動時自動通知 Jenkins 2 進行編譯建置 (GitHub Plugin)"
date: 2017-02-11T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps","Git"]
slug: "github-plugin-jenkins-webhook"
aliases:
    - /2017/02/github-plugin-jenkins-webhook.html
---
## GitHub Repository 有變動時自動通知 Jenkins 2 進行編譯建置 (GitHub Plugin)

之前筆記 [遠端 Git Repository 有 merge 時自動通知 Jenkins 2 進行編譯建置 (webhook)](/git-repository-jenkins2-webhook) 紀錄了如何以 GitHub 為例設定 Git server 提供的 webhook 功能與 Jenkins 2 建置自動化建置的 CI 環境，其中 GitHub 的相關功能 Jenkins 2 有相對應的 Plugin 功能，今天就來看看設定上有什麼不同

## 文章大綱

1. 安裝 GitHub Plugin
2. 設定 GitHub Plugin
3. 設定 build project
4. 設定 GitHub
5. 結果

## 安裝 GitHub Plugin

1. Manage Jenkins --> Manage Plugins

    ![1install](https://cloud.githubusercontent.com/assets/3851540/21919841/52c569f6-d998-11e6-91aa-c789cf9619be.png)

2. `Available` tab --> Filter `GitHub plugin` --> 勾選 `GitHub plugin`

    ![2github](https://cloud.githubusercontent.com/assets/3851540/21919842/52c84090-d998-11e6-96dc-e71c1944d799.png)

## 設定 GitHub Plugin

1. Manage Jenkins --> Configure System

    ![3globalconfig](https://cloud.githubusercontent.com/assets/3851540/21919843/52ca3da0-d998-11e6-8552-090862f6dbf1.png)

2. ADD GITHUB SERVER

    ![4addgitserver](https://cloud.githubusercontent.com/assets/3851540/21919844/52e3b06e-d998-11e6-9ecd-96faeaaa2c04.png)

    ![5addgitserver](https://cloud.githubusercontent.com/assets/3851540/21919846/52e9929a-d998-11e6-8afb-ef58a8ce03aa.png)
3. ADD Credential

    ![6addcrend](https://cloud.githubusercontent.com/assets/3851540/21919845/52e5febe-d998-11e6-92bd-704b8f84b82a.png)
4. Credential detain

    ![7cred](https://cloud.githubusercontent.com/assets/3851540/21919848/52ecaed0-d998-11e6-92f6-b4b0ceadc07d.png)
    - Kind --> `Secret text` (由 Plain Credentials Plugin 提供)
    - Secret (由 GitHub 產生)
        - GitHub 個人設定

            ![8githubsetting](https://cloud.githubusercontent.com/assets/3851540/21919847/52eb7268-d998-11e6-881d-8ed2b97b370d.png)
        - Personal access tokens --> Generate new token

            ![9gentoken](https://cloud.githubusercontent.com/assets/3851540/21919837/52c1529e-d998-11e6-88a6-3d2d74d7d589.png)
        - New personal access token
            - 填寫 Token 描述
            - 選擇 scopes <span style="color:red">需要 `admin:org_hook`</span>

                ![10tokeninfo](https://cloud.githubusercontent.com/assets/3851540/21919839/52c339d8-d998-11e6-81bd-db10793fb610.png)
        - Personal access tokens
            - <span style="color:red">只有出現一次</span>

                ![11gettoken](https://cloud.githubusercontent.com/assets/3851540/21919838/52c2fe6e-d998-11e6-8d8c-583b67b6c7fe.png)

## 設定 build project

- GitHub project
  - 填入 Project url

        ![12projecturl](https://cloud.githubusercontent.com/assets/3851540/21987933/683c3544-dc40-11e6-8dfd-281a5f7e6d2b.png)

- Source Code Management
  - Git
  - 填寫 Git Repository 資訊

        ![13gitrepo](https://cloud.githubusercontent.com/assets/3851540/21987931/683822b0-dc40-11e6-9b1a-2be3bf1bb2d4.png)
- GitHub hook trigger for GITScm polling

    ![0githubhook](https://cloud.githubusercontent.com/assets/3851540/22008497/af30e6c2-dcb6-11e6-91c2-515077ac9322.png)
  - ***有安裝 GitHub plugin 才會出現這個選項***

## 設定 GitHub

1. Add service
   - Settings --> Integration & services --> Add service --> "Jenkins"  --> Jenkins(GitHub plugin)

        ![13githubservice](https://cloud.githubusercontent.com/assets/3851540/21987928/68368a54-dc40-11e6-904d-d6d0b60cb48a.png)

2. 填寫 Jenkins service hook url
    - 會自動轉導至 Settings --> Webhooks
    - 填入 `Jenkins hook url`
        - Jenkins 對外網址加上 `/github-webhook`
    - 確認 `Active`

        ![14serviceinfo](https://cloud.githubusercontent.com/assets/3851540/21987929/6837c824-dc40-11e6-8d91-f3058b60e905.png)

## 結果

1. 收到通知

    ![15received](https://cloud.githubusercontent.com/assets/3851540/21987930/6837cd06-dc40-11e6-829b-d5cdd2bce37a.png)

2. 完成 build

    ![16done](https://cloud.githubusercontent.com/assets/3851540/21987932/6838d2dc-dc40-11e6-96c2-e708fb2f33b0.png)

## 參考資訊

1. [GitHub Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin)
2. [How to Start Working with the GitHub Plugin for Jenkins](https://www.blazemeter.com/blog/how-start-working-github-plugin-jenkins)
3. [遠端 Git Repository 有 merge 時自動通知 Jenkins 2 進行編譯建置 (webhook)](/git-repository-jenkins2-webhook)
