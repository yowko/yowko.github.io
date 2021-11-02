---
title: "Jenkins 如何在 Pipeline 中平行 build Job 與失敗不中斷 job"
date: 2017-04-26T02:21:00+08:00
lastmod: 2021-11-02T02:21:42+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins-fail-and-continue"
aliases:
    - /2017/04/jenkins2-fail-and-continue.html
    - /2017/04/jenkins-fail-and-continue
---
## Jenkins 如何在 Pipeline 中平行 build Job 與失敗不中斷 job

同事打算設定固定周期性執行 pipeline job 將所有專案 build 一輪，讓專案可以常保在健康的狀態。

非常正確的想法，但我的疑問是為什麼需要使用到 pipeline job：因為 job 有互相依賴的狀況，得確定基底 job 完成才能執行其他 job，就讓我們來看可以怎麼設定

## 使用 Groovy plugin 進行設定

> 安裝 Groovy plugin 的方式請參考 [Jenkins 2 如何取得上次編譯成功的時間](/jenkins2-getlastsuccessfulbuild)

## 同時 build 多個 stage

1. 原本循序做法

    * 語法

        ```groovy
        stage('testPowerShell') {
            build job: 'testPowerShell', parameters: [
            [$class: 'StringParameterValue', name: 'serviceNames', value: 'reveal,TestCert'],
            [$class: 'StringParameterValue', name: 'filelocation', value: 'D:\\serverlist.json'],
            [$class: 'StringParameterValue', name: 'groupNo', value: '0']
            ]
        }
        stage('TestCopy') {
            build job: 'TestCopy', parameters: [
                [$class: 'StringParameterValue', name: 'excludeFolders', value: 'D:\\Downloads\\NewReveal\\reveal.js-master\\test D:\\Downloads\\NewReveal\\reveal.js-master\\js']
            ]
        }
        ```

    * 依序執行

        ![1step1](https://cloud.githubusercontent.com/assets/3851540/25400369/e554f71e-2a24-11e7-932c-c341725b19a0.png)

        ![2step2](https://cloud.githubusercontent.com/assets/3851540/25400370/e5597334-2a24-11e7-8458-d11c8a4fc2a1.png)

    * 有錯直接中斷後續動作

        ![3blocking](https://cloud.githubusercontent.com/assets/3851540/25400372/e55cf770-2a24-11e7-87c5-a68fa110f166.png)

2. 使用 `parallel` 平行處理

    * 語法

        ```groovy
        parallel(
            job1:{stage('testPowerShell') {
                build job: 'testPowerShell', parameters: [
                [$class: 'StringParameterValue', name: 'serviceNames', value: 'reveal,TestCert'],
                [$class: 'StringParameterValue', name: 'filelocation', value: 'D:\\serverlist.json'],
                [$class: 'StringParameterValue', name: 'groupNo', value: '0']
                ]
            }},
            job2:{stage('TestCopy') {
            build job: 'TestCopy', parameters: [
                [$class: 'StringParameterValue', name: 'excludeFolders', value: 'D:\\Downloads\\NewReveal\\reveal.js-master\\test D:\\Downloads\\NewReveal\\reveal.js-master\\js']
            ]
            }}
        )
        ```

    * 平行執行

        > 將多個 stage 視為同一個 node 會同時啟動，會等待較長的 stage 完成後才統一回傳結果

        ![4starttoget](https://cloud.githubusercontent.com/assets/3851540/25400371/e55c8e98-2a24-11e7-98b7-e1c07408b795.png)

        ![5stoptoget](https://cloud.githubusercontent.com/assets/3851540/25400368/e5346f12-2a24-11e7-9fce-ebd049f812d9.png)

    * 出錯仍會執行其他 stage

        ![6noblocking](https://cloud.githubusercontent.com/assets/3851540/25400367/e53343bc-2a24-11e7-885c-015e3d8236ac.png)

## 在同一個 stage build 多個 job

這比較符合同事的需求，需要有基礎的 job 先完成後再進行其他 job 的建置

* 語法

    > 只在 stage 2 去載入多個 job 並使用 parallel build

    ```groovy
    def transformIntoStep(jobFullName) {
        return {
            build job: jobFullName,
            parameters: [string(name: 'Environ', value: 'QA')],
            quietPeriod: 3
            }
    }
                
    stage('Step1') {
        build job: 'A01-TestDev', parameters: [
        [$class: 'StringParameterValue', name: 'serviceNames', value: 'reveal,TestCert'],
        [$class: 'StringParameterValue', name: 'filelocation', value: 'D:\\serverlist.json'],
        [$class: 'StringParameterValue', name: 'groupNo', value: '0']
        ]
    }
    stage('Step2') {
        def branches = [: ]
        def jobs = "${buildjobs}".tokenize(',')
        def i = 0;
        for (jobName in jobs) {
        //println(jobName);
        branches["job${i}"] = transformIntoStep(jobName);
        i++;
        }
        //println(branches);
        parallel branches;
                
    }
    stage('Step3') {
        build job: 'A01-TestDev', parameters: [
        [$class: 'StringParameterValue', name: 'serviceNames', value: 'reveal,TestCert'],
        [$class: 'StringParameterValue', name: 'filelocation', value: 'D:\\serverlist.json'],
        [$class: 'StringParameterValue', name: 'groupNo', value: '0']
        ]
    }
    ```

* 正常執行

    ![7pipelineparallel](https://cloud.githubusercontent.com/assets/3851540/25400364/e531b9d4-2a24-11e7-9961-76f010f609c3.png)

    ![8parallelok](https://cloud.githubusercontent.com/assets/3851540/25400366/e533326e-2a24-11e7-83d6-706d370c8dc8.png)

* 某個 job 有異常，也不會影響其他 job build

    ![9parallelfail](https://cloud.githubusercontent.com/assets/3851540/25400363/e53178e8-2a24-11e7-84fd-74205e439893.png)

    ![10paralleljobok](https://cloud.githubusercontent.com/assets/3851540/25400365/e53300b4-2a24-11e7-9e9a-c4c60411e6a9.png)

## 心得

我測試下來，塞入多個 build job 的行為如果不拉出去用 function (transformIntoStep)來處理，會只 build 最後一個 job 多次，效果跟 [Jobs In Parallel](https://jenkins.io/doc/pipeline/examples/#jobs-in-parallel) 相同，文件有提到 build array 是拿來 build 同一個 job 多次用的，這個要多注意

## 參考資料

1. [Pipeline Examples](https://jenkins.io/doc/pipeline/examples/)
2. [Parallelism and Distributed Builds with Jenkins](https://www.cloudbees.com/blog/parallelism-and-distributed-builds-jenkins)
3. [Jenkins 2 如何取得上次編譯成功的時間](/jenkins2-getlastsuccessfulbuild)
4. [Jobs In Parallel](https://jenkins.io/doc/pipeline/examples/#jobs-in-parallel)
