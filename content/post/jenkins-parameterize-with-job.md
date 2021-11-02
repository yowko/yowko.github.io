---
title: "Jenkins 將其他 job 名稱變成可選擇的參數"
date: 2017-03-13T02:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins-parameterize-with-job"
aliases:
    - /2017/03/jenkins2-parameterize-with-job.html
---
## Jenkins 將其他 job 名稱變成可選擇的參數

專案愈來愈多，加上最近正在更新專案的 Jenkins，正好趁著這次機會將各個專案的 CI 流程統一，以往新增專案就會手動為專案加上一個 Build job，一個一個加其實沒什麼感覺，但需要一次做調整時就快起肖了。所以想要使用模組化的 build flow，今天就先來介紹如何將 job name 當做 build job 的參數

## 安裝 plugin

1. Manage Jenkins --> Manage Plugins

    ![1plugin](https://cloud.githubusercontent.com/assets/3851540/23693986/5a47f078-0412-11e7-892b-0fcce88f3abd.png)

2. Available tab --> Search "Extended Choice Parameter" --> 勾選 "Extended Choice Parameter plugin" --> install

    ![2install](https://cloud.githubusercontent.com/assets/3851540/23693989/5a69b550-0412-11e7-9e52-80bed6ea42be.png)

## 設定 plugin

1. This porject is paramerterized

    ![3paramertized](https://cloud.githubusercontent.com/assets/3851540/23693988/5a666db4-0412-11e7-915a-92b145dfc9d7.png)

2. Add Extended Choice Parameter --> Basic Parameter Types

    ![4extented](https://cloud.githubusercontent.com/assets/3851540/23693987/5a65d5a2-0412-11e7-9d3b-bb711a289ab0.png)

3. Parameter Type

    ![5paramtype](https://cloud.githubusercontent.com/assets/3851540/23693981/5a40be8e-0412-11e7-9109-994eff72094f.png)
    - 填寫顯示的數量

4. Choose Source for Value
    - Groovy Script

        ```groovy
        def jobs = jenkins.model.Jenkins.instance.getJobNames()
        def matchjobs = jobs.findAll{ name -> name =~ /(A|B|C)\d{2}.*/ }
        return matchjobs
        ```

    - 使用 Groovy Script 來過濾 jobname
    - 這邊使用 regular expression 來過濾

        ![6sourcevalue](https://cloud.githubusercontent.com/assets/3851540/23693982/5a42111c-0412-11e7-8fe9-bf905341efc2.png)

## 啟用 script 執行

1. Build with Parameters

    ![7buildwith](https://cloud.githubusercontent.com/assets/3851540/23693984/5a44982e-0412-11e7-974c-c0620589ed04.png)

2. Appove Groovy script

    ![8approve](https://cloud.githubusercontent.com/assets/3851540/23693983/5a42871e-0412-11e7-82d8-d09b5b47ac0c.png)

## 實際效果

![9result](https://cloud.githubusercontent.com/assets/3851540/23693985/5a44be1c-0412-11e7-8b52-210aef6234b5.png)

## 心得

經過上面的設定，就可以將現有的 job 拿來當做 build 參數，適合接在 pipeline job 後面，做後續的動作，這也是 build job 模組化的第一步

## 參考資訊

1. [Extended Choice Parameter plugin](https://wiki.jenkins-ci.org/display/JENKINS/Extended+Choice+Parameter+plugin)
