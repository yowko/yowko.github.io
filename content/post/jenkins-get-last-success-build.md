---
title: "Jenkins 如何取得上次編譯成功的時間"
date: 2017-04-07T23:30:00+08:00
lastmod: 2018-09-16T23:30:04+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins-get-last-success-build"
aliases:
    - /2017/04/jenkins2-getlastsuccessfulbuild.html
    - /2017/04/jenkins-get-last-success-build
---
# Jenkins 如何取得上次編譯成功的時間
之前曾經在 [Jenkins 2 將其他 job 名稱變成可選擇的參數](//blog.yowko.com/2017/03/jenkins2-parameterize-with-job.html) 介紹過該如何把其他 job name 當做參數，經過一段時間的發展，job 默默地成長到 70-80 個，已經很難一眼就找到所需 job 的地步，所以同事給了新的需求，將上次成功編譯時間是今天的 job 預設勾選，降低人為漏看的機會

## 如何取得上次編譯成功的時間

*   groovy 語法如下
    
    ```groovy
    import jenkins.model.Jenkins
    def item = Jenkins.instance.getItem("jobname")//jobname 請換成你實際的 job name
    def  lastSuccess=item.getLastSuccessfulBuild().getTime()
    println "${lastSuccess}"
    ```

*   建議可以使用 build step 來進行開發及 debug


1.  安裝 groovy plugin
    
    ![1manageplugin](https://cloud.githubusercontent.com/assets/3851540/24797673/e0666cba-1bc4-11e7-8a5d-fc2785585ed9.png)

    ![2installgroovy](https://cloud.githubusercontent.com/assets/3851540/24797674/e0804ff4-1bc4-11e7-91f8-274fe6c81cb6.png)

2.  在 build step 測試語法

    ![3buildStep](https://cloud.githubusercontent.com/assets/3851540/24797675/e0812ae6-1bc4-11e7-8b2f-29f4395b1443.png)

    ```groovy
    import jenkins.model.Jenkins
    def item = Jenkins.instance.getItem("A01-TestDev")
    def  lastSuccess=item.getLastSuccessfulBuild().getTime()
    println "${lastSuccess}"
    ```

3.  結果

    ![4getsuccdatetime](https://cloud.githubusercontent.com/assets/3851540/24797676/e0823b3e-1bc4-11e7-9d6a-4dd7604fd949.png)

## 預設勾選

這個部份 Extended Choice Parameter plugin 已經為我們設想到了，在原本設定 source value 的下方就提供 source for default value 的設定，設定方式與 source for value 相同，一樣可以使用 groovy

![5defaultvalue](https://cloud.githubusercontent.com/assets/3851540/24797668/e05e2262-1bc4-11e7-9108-297b7129716b.png)

1.  Choose Source for Default Value

    > 這邊使用 inline groovy script (Default Groovy Script)

    ```groovy
    import jenkins.model.Jenkins
    def result= []
    def jobs = jenkins.model.Jenkins.instance.getJobNames()
    def matchjobs = jobs .findAll{ name -> name =~  /(A|B|C)\d{2}.*/ }
    matchjobs.each { 
        def today = new Date().format("yyyyMMdd")
        def item = Jenkins.instance.getItem("${it}")
        def  lastSuccess=item.getLastSuccessfulBuild().getTime().format("yyyyMMdd")
        def output=today.equals(lastSuccess)
        if(output)
        {
            result.add("${it}")
        }
    }
    return result
    ```

    ![10defaultsetting](https://cloud.githubusercontent.com/assets/3851540/24797770/5cf8db00-1bc5-11e7-9acf-2e4b05964822.png)

2.  加入 groovy 後，記得要同意執行 groovy script
    *   Build with Parameters 會出現需要同意提示
        
        ![6needapprove](https://cloud.githubusercontent.com/assets/3851540/24797669/e05f01d2-1bc4-11e7-800c-6da7ebc32efc.png)

    *   Approve

        ![7apporve](https://cloud.githubusercontent.com/assets/3851540/24797670/e05f5e5c-1bc4-11e7-87e2-13da794a1926.png)

3.  前後對照
    *   設定前
        
        ![9BEFORE](https://cloud.githubusercontent.com/assets/3851540/24797672/e06622a0-1bc4-11e7-9d25-98ec3add97d1.png)

    *   設定後
        
        ![8AFTER](https://cloud.githubusercontent.com/assets/3851540/24797671/e05feb74-1bc4-11e7-996b-d2fdaa967247.png)

# 參考資訊
1.  [Jenkins - groovy script - get last successful build date in dd-mm-yyyy format](http://stackoverflow.com/questions/32180785/jenkins-groovy-script-get-last-successful-build-date-in-dd-mm-yyyy-format)
2.  [How to compare two Dates without the time portion?](http://stackoverflow.com/questions/1439779/how-to-compare-two-dates-without-the-time-portion)
