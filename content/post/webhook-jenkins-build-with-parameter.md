---
title: "Git webhook 如何驅動 Jenkins 的參數化建置 (Build with Parameters)"
date: 2017-07-05T22:24:00+08:00
lastmod: 2020-12-11T22:24:00+08:00
draft: false
tags: ["Git","Jenkins"]
slug: "webhook-jenkins-build-with-parameter"
aliases:
    - /2017/07/webhook-jenkins-build-with-parameter.html
---
# Git webhook 如何驅動 Jenkins 的參數化建置 (Build with Parameters)
之前文章 [Jenkins 2 將其他 job 名稱變成可選擇的參數](/2017/03/jenkins2-parameterize-with-job.html)、[Jenkins 2 如何建立 Pipeline job](/2017/02/jenkins-2-pipeline-job.html) 曾經簡單地介紹過模組化 build flow 的想法，透過參數化建置 (Build with Parameters) 達成使用相同 job 執行不同 project 的建置

但這樣的做法在設定 webhook 執行 Continuous integration - CI build 時需要額外在 Git Server 的 Repository 上動些手腳，就來看看該如何設定

## Jenkins Job 基本設定

1.  有個名為 `JobName` 的 String Parameter

    ![1stringparam](https://user-images.githubusercontent.com/3851540/27852084-4f3b3bd2-6190-11e7-96fd-1a5b0541f79a.png)

2.  記得設定 Source Code Management
3.  Build Triggers 有 Trigger builds remotely (e.g., from scripts)

    ![2buildtrigger](https://user-images.githubusercontent.com/3851540/27852085-4f52aa1a-6190-11e7-8e30-c77d35c2d48e.png)

4.  簡單的印出上述參數

    ![3buildprint](https://user-images.githubusercontent.com/3851540/27852086-4f5475b6-6190-11e7-833d-9d03cdeea1b8.png)

## Jenkins 2 上的 webhook 設定

這邊就不重複贅述，請大家直接參考之前文章 [遠端 Git Repository 有 merge 時自動通知 Jenkins 2 進行編譯建置 (webhook)](/2017/02/git-repository-jenkins2-webhook.html)

*   需要留意的是 Jenkins 上關於 webhook 的說明

    ```
    Use the following URL to trigger build remotely: JENKINS_URL/job/A01-TestDev/build?token=TOKEN_NAME or /buildWithParameters?token=TOKEN_NAME
    Optionally append &cause=Cause+Text to provide text that will be included in the recorded build cause.
    ```

## Git Server 的 webhook 設定

> 今天會使用 gitlab 做為範例

*   Git Repository --> Setting --> Integrations

    ![4setting](https://user-images.githubusercontent.com/3851540/27852087-4f57108c-6190-11e7-8d21-a2500db9b47e.png)

*   URL
    *   pattern

        ```
        http://{jenkins_Url}/job/{job_name}/buildWithParameters?token={token}&{ParameterName}={ParameterValue}
        ```
    *   範例

        ```
        http://127.0.0.1/job/A02-TestQA/buildWithParameters?token=39296d67-6750-45ab&JobName=fromGitLab
        ```
        *   如果執行 webhook 時有 Jenkins 權限上問題請試試使用 API TOKEN


            *   Jenkins People --> User Id

                ![6token1](https://user-images.githubusercontent.com/3851540/27852089-4f602a82-6190-11e7-9a0d-fbc7fa9d83e1.png)

            *   Configure --> API Token

                ![6token2](https://user-images.githubusercontent.com/3851540/27852079-4f2fa11e-6190-11e7-9c73-a62fd0397f5a.png)

                ![6token3](https://user-images.githubusercontent.com/3851540/27852082-4f353a84-6190-11e7-86a7-5152e5a1a91a.png)

            *   用法
                
                ```
                http://{user_id}:{token}@{jenkins_Url}/job/{job_name}/buildWithParameters?token={token}&{ParameterName}={ParameterValue}
                ```
    *   實測下 URL 中的 token 如果太長無法正確執行

*   Trigger

    > 請依實際使用的 trigger 來挑選

*   SSL verification

    > 預設啟用 SSL，沒有 SSL 記得取消

## 設定完成後可以測試執行

*   執行測試

    ![7test](https://user-images.githubusercontent.com/3851540/27852081-4f33cfb4-6190-11e7-9a86-ad9122fa9067.png)

*   成功發出 webhook

    ![8testok](https://user-images.githubusercontent.com/3851540/27852080-4f315266-6190-11e7-88b2-747b23a3ab2a.png)

*   Jenkins 執行 build

    ![9jenkinsbuild](https://user-images.githubusercontent.com/3851540/27852083-4f3852aa-6190-11e7-8002-9da68d2c6d06.png)

*   建置結果

    ![5result](https://user-images.githubusercontent.com/3851540/27852088-4f5b267c-6190-11e7-9f55-a1c114380e7f.png)

## 心得

原本需要參數的 job 會直接使用參數預設值來進行建置而造成結果不如預期，但透過 webhook 的 url 設定就可以傳遞參數給本需要參數才能建置的 job 直接進行 build 這樣一來就不用為了 webhook 修改原本的 build job，讓模組化 CI 更往前了一步

# 參考資訊

1.  [Jenkins 2 將其他 job 名稱變成可選擇的參數](/2017/03/jenkins2-parameterize-with-job.html)
2.  [Jenkins 2 如何建立 Pipeline job](/2017/02/jenkins-2-pipeline-job.html)
3.  [遠端 Git Repository 有 merge 時自動通知 Jenkins 2 進行編譯建置 (webhook)](/2017/02/git-repository-jenkins2-webhook.html)
