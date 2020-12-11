---
title: "Jenkins Job 觸發其他需要參數的 Job"
date: 2017-07-06T22:17:00+08:00
lastmod: 2020-12-11T22:17:00+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins-job-trigger-paramerized-job"
aliases:
    - /2017/07/jenkins2-job-trigger-paramerized-job.html
    - /2017/07/jenkins-job-trigger-paramerized-job/
---
# Jenkins Job 觸發其他需要參數的 Job
Jenkins 完成專案 Continuous integration - CI build 後只能確保該專案可以通過建置，但系統各個功能是不是可以如預期執行有時是需要多個專案共同搭配的結果

同事遇到的問題就是這個情境：A 專案 job 建置完成後，需要 B job 執行後續動作加上 B job 需要在 build 時提供參數才能正確執行。之前的文章中並沒有紀錄到這種情境，剛好同事問起，順手紀錄一下

## 基本設定說明

1.  `A01-TestDev` 是一般 project job

    > 用來模擬一般 project 的 free-style build job

2.  `TestTriggerPipeline` 則是 Pipeline job

    > 用來模擬 project 成功建置後執行 stop iis、deploy、start iis 的動作

    *   有一個名為 `jobNames` 的 String Parameter 用來接收輸入的參數

        ![3strigparameter](https://user-images.githubusercontent.com/3851540/27906371-d6bd1b08-6275-11e7-9787-6e889b69e892.png)

    *   Pipeline 內容

        > 以下使用簡化內容模擬，如果想多了解 Pipeline job 可以參考 [Jenkins 2 如何建立 Pipeline job](/2017/02/jenkins-2-pipeline-job.html)

        *   Definition 使用 Pipeline script
        *   script 直接列出 傳入的 jobNames

            > `println "This job was caused by " + "${jobNames}"`

3.  `A01-TestDev` 成功後接著執行 `TestTriggerPipeline`

    > `TestTriggerPipeline` 需要參數 - jobNames 來決定該怎麼進行後續步驟

## 安裝 Plugin - Parameterized Trigger Plugin

> `Parameterized Trigger Plugin` 可以讓我們在觸發其他 job 時傳遞參數

1.  Jenkins 主畫面 Manage Jenkins --> Manage Plugins

    ![1managejenkins](https://user-images.githubusercontent.com/3851540/27906369-d6b8a03c-6275-11e7-9562-2c4cedb00c5c.png)

2.  Available tab --> search `Parameterized Trigger Plugin` --> 勾選 `Parameterized Trigger Plugin` --> install

    ![2install](https://user-images.githubusercontent.com/3851540/27906368-d6b75132-6275-11e7-8b6f-92f9b83b3cb6.png)

## 設定 `A01-TestDev` Job

1.  在 `Post-build Actions` 加上 `Trigger parameterized build on other projects`

    ![4addtrigger](https://user-images.githubusercontent.com/3851540/27906372-d6d7fbc6-6275-11e7-9bec-73e6022c2d65.png)

2.  設定 `Trigger parameterized build on other projects`

    ![5edittrigger](https://user-images.githubusercontent.com/3851540/27906363-d68f31c0-6275-11e7-8d69-a968dbe18104.png)

    *   Projects to build

        > 有自動提示

        ![6projecthint](https://user-images.githubusercontent.com/3851540/27906362-d68cf220-6275-11e7-85b0-87f8566c8ec2.png)

    *   Trigger when build is

        > 視情境調整

        ![7projectstatus](https://user-images.githubusercontent.com/3851540/27906366-d696801a-6275-11e7-8520-93bb413cea50.png)

3.  加上 Parameter

    *   視情境使用 `Current build parameters` 或是 `Predefined parameters`

        ![8addparam](https://user-images.githubusercontent.com/3851540/27906364-d68fe08e-6275-11e7-905e-ccecaacf4222.png)

4.  設定 Parameter

    *   設定目標 job 所需參數的值

        > 範例是將 `jobNames` 指定為目前的 job name

        ![9passjobname](https://user-images.githubusercontent.com/3851540/27906365-d690ac1c-6275-11e7-86c4-eadf1f3b2287.png)

5.  完整設定

    > `A01-TestDev` Job build success 後使用當前 job name 當做 jobNames 參數直接執行 `TestTriggerPipeline`

    ![10postbuildsetting](https://user-images.githubusercontent.com/3851540/27906367-d6b68234-6275-11e7-93c8-0b757a18261d.png)

## 實際效果

![11result](https://user-images.githubusercontent.com/3851540/27906370-d6b92494-6275-11e7-966b-4e4298f2c095.png)

可以看到 build 是由 `upstream project A01-TestDev` 發動的

## 心得

原本想透過 groovy 來取得 `upstream` 資訊，嘗試了好久一直找不到方法，後來透過目前方式不僅設定容易，也少了維護 groovy 的成本，真是不賴

# 參考資訊

1.  [Jenkins - How to get and use upstream info in downstream](http://techqa.info/programming/question/39207924/jenkins---how-to-get-and-use-upstream-info-in-downstream)
