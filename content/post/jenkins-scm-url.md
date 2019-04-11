---
title: "如何透過 jobname 取得 Jenkins 2 的 SCM Repository URL 及如何取得 BUILD_ID 及 BUILD_NUMBER 基本語法筆記 2"
date: 2017-03-14T02:43:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins-scm-url"
aliases:
    - /2017/03/jenkins2-scm-url.html
    - /2017/03/jenkins-scm-url/
---
# 如何透過 jobname 取得 Jenkins 2 的 SCM Repository URL 及如何取得 BUILD_ID 及 BUILD_NUMBER
公司的 CI 流程需要使用 SCM Repository URL 來進行不同環境間的程式碼切換，原本為每個 job 都加上一個 URL 設定，但這樣一來就需要有人去維護這個設定，經年累月後物換星移後，默默地增加了可能出錯的一個隱形問題，最好的方式就是使用 SCM Repository URL，而為了配合模組化建置的計畫，就需要有個 build template 然後使用參數來進行資料的取得及後續作業

## 1. 如何取得 SCM Repository URL
- 讀取 job config
    
    Jenkins 的 job 設定都是個 xml 檔，只要使用 xml 的處理方式，就可以輕鬆取得 job 上的任何設定了，接著就來看看可以怎麼處理(以下會使用 PowerShell 來示範)
- config 位置
     job 的 config 就位於 "{Jenkins 安裝目錄}\jobs\{jobname}\config.xml"
    
    > e.g. `C:\Program Files (x86)\Jenkins\jobs\A01\config.xml`
- PowerShell 語法
     
     ```ps1
     #將 config 檔案 parse 成 xml 物件
     [xml]$xmldata = get-content "{Jenkins 安裝目錄}\Jenkins\jobs\{jobname}\config.xml"
     #依階層取值
     ```

#### GIT
- Jenkins 的設定
    
    ![1gitsource](https://cloud.githubusercontent.com/assets/3851540/23746390/b6da1ad4-04f6-11e7-9cb6-e3229fb841c5.png)
- config 內容
    
    ![3gitconfig](https://cloud.githubusercontent.com/assets/3851540/23746391/b6db35c2-04f6-11e7-81dd-07d84777a8c3.png) 
- 使用 powershell 取得 url 設定
    
    ```ps1
    [xml]$xmldata = get-content "{Jenkins 安裝目錄}\Jenkins\jobs\{jobname}\config.xml"
    $xmldata.project.scm.userRemoteConfigs.'hudson.plugins.git.UserRemoteConfig'.url
    ```
#### SVN
- Jenkins 的設定
    
    ![2scmsource](https://cloud.githubusercontent.com/assets/3851540/23746389/b6d74390-04f6-11e7-940a-08e5d3615b81.png) 
- config 內容
    
    ![4svnconfig](https://cloud.githubusercontent.com/assets/3851540/23746392/b6e01b28-04f6-11e7-847c-7a7c804531b4.png) 
- 使用 powershell 取得 url 設定
    
    ```ps1
    [xml]$xmldata = get-content "{Jenkins 安裝目錄}\Jenkins\jobs\{jobname}\config.xml"
    $xmldata.project.scm.locations.'hudson.scm.SubversionSCM_-ModuleLocation'.remote
    ```
## 2. 如何取得 BUILD_ID and BUILD_NUMBER
依 [Environment Variables BUILD_ID and BUILD_NUMBER now return the same value](https://issues.jenkins-ci.org/browse/JENKINS-26520) 揭露 BUILD_ID 與 BUILD_NUMBER 目前會是相同的
- BUILD_ID 
    
    >`$env:BUILD_ID` 
- BUILD_NUMBER
    
    >`$env:BUILD_NUMBER`

# 參考資訊
1. [Getting job repository URL in Jenkins build plugin](http://stackoverflow.com/questions/19359118/getting-job-repository-url-in-jenkins-build-plugin)
2. [Environment Variables BUILD_ID and BUILD_NUMBER now return the same value](https://issues.jenkins-ci.org/browse/JENKINS-26520)

