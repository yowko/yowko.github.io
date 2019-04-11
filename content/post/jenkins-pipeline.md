---
title: "Jenkins 2 如何建立 Pipeline job"
date: 2017-02-22T01:42:34+08:00
lastmod: 2018-09-12T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps"]
slug: "jenkins-pipeline"
aliases:
    - /2017/02/jenkins-2-pipeline-job.html
---
# Jenkins 2 如何建立 Pipeline job
Pipeline 是 Jenkins 2 的一大特色，其概念就是將建置流程步驟透過 DSL(Domain-Specific Languages) 來定義與描述，使用的語法是 groovy，但如果只為了 build job 而刻意學 goovy 個人擔心學完不常用很快就忘，實際上我只用基本語法來完成 Pipeline 的工作，因此本文關於 groovy 語法的部份只會帶過而已。

本文的另個重點，將會使用 Jenkins 2 搭配 PowerShell 與 robocopy 來完成持續性部署 (Continuous Deployment) 的自動化作業。

## 使用情境
1. checkout source code and build
2. 停止網站 or windows service
3. 程式部署
4. 啟動網站 or windows service

## 實際做法
1. checkout source code and build
    - 可以參考 [如何使用 Jenkins 2 建置 .NET 專案](http://blog.yowko.com/2017/02/jenkins-2-build-dotnet-project.html) 來建立 build job, 這邊就不重複贅述
    - 需要注意的是 build 參數要加上 publish 設定後面才可以進行部署
        
        ```
        /p:DeployOnBuild=True /p:DeployDefaultTarget=WebPublish /p:WebPublishMethod=FileSystem /p:DeleteExistingFiles=True /p:publishUrl={filepath}
        ```
2. 停止網站 or windows service
    - 建立 StopWebSite 的 freestyle project
        
        ![1stopwebsite](https://cloud.githubusercontent.com/assets/3851540/23155475/a52e8b68-f84d-11e6-917c-0bce2431dc8e.png) 
    - 使用參數化建置
        
        ![2addparameterize](https://cloud.githubusercontent.com/assets/3851540/23155477/a53cb65c-f84d-11e6-812f-b959cee8e16c.png) 
        - 加入 string 參數來表示目標 server
            
            ![4serverip](https://cloud.githubusercontent.com/assets/3851540/23155480/a552f232-f84d-11e6-9faa-3bce3f7012f4.png) 
        - 加入 string 參數來表示欲停止的 IIS 網站名稱
            
            ![3sitename](https://cloud.githubusercontent.com/assets/3851540/23155479/a5483c16-f84d-11e6-8b62-758fc4a2a259.png)
    - 使用 PowerShell 來停止 website
        
        ![5ps](https://cloud.githubusercontent.com/assets/3851540/23155470/a4ed669c-f84d-11e6-8113-15045775ea85.png)
        - 記得安裝 PowerShell plugin (沒裝過的可以參考 [Jenkins 2 如何使用 PowerShell 以及自定 build fail (指定 exit code)](https://blog.yowko.com/2017/02/jenkins2-powershell-plugin.html))
        - 加入一個 build step 使用 Windows PowerShell
            
            ```ps1
            if ( Invoke-Command -ComputerName  $env:serverIP -ScriptBlock { (Get-WebAppPoolState -Name $args[0]).Value -eq "Started" } -argumentlist $env:siteName)
            {
                Invoke-Command -ComputerName $env:serverIP -ScriptBlock { Stop-WebAppPool -Name $args[0] } -argumentlist $env:siteName
            }
            
            do
            {
                Invoke-Command -ComputerName $env:serverIP -ScriptBlock { Stop-WebAppPool -Name $args[0] } -argumentlist $env:siteName
                Start-Sleep -Seconds 1
            }
            until ( Invoke-Command -ComputerName  $env:serverIP -ScriptBlock { (Get-WebAppPoolState -Name $args[0]).Value -eq "Stopped" } -argumentlist $env:siteName  )
            ```
        - 在 Windows PowerShell 要取用定義在 Jenkins 的變數，記得在 變數前面加上 `$env:`
        
3. 程式部署
    - 建立 DeployWebSite 的 freestyle project
    - 使用參數化建置
        
        ![6deploy](https://cloud.githubusercontent.com/assets/3851540/23155471/a517c63a-f84d-11e6-8184-5ec8647ed054.png) 
        - 加入 string 參數來表示部署`來源`位置
        - 加入 string 參數來表示部署`目標`位置
    - 使用 PowerShell 與 robocopy 來複製檔案
        
        ```ps1
        Try
        {
            $lasterror=robocopy $env:SourceFolder $env:DestinationFolder *.* /E /MT /purge
        }
        Catch
        {
            write-output "get data fail!"
            exit 1
        }
        if($lasterror -le 7)
        {
         exit 0
        }
        ```
     - 需要注意的是 robocopy 的 exit code
     
        exit code|	說明
        :---|:---
        0|	沒有錯誤，也沒有複製任何檔案，來源與目標已同步
        1|	檔案已成功複製
        2|	偵測到目標有額外檔案(檔案比來源目錄還多)
        4|	檔案或是資料夾不一致
        8|	檔案或是資料夾無法被複製(複製出錯且重試次數過多)
        16|	錯誤，無法複製任何檔案(使用方法錯誤或是權限不足)

4. 啟動網站 or windows service
    - 建立 StartWebSite 的 freestyle project
    - 使用參數化建置
        - 加入 string 參數來表示目標 server
        - 加入 string 參數來表示欲停止的 IIS 網站名稱
    - 使用 PowerShell 來啟動 website
    - 步驟跟  `停止網站 or windows service` 一樣，只是換成啟動
        
        ```ps1
        if ( Invoke-Command -ComputerName  $env:serverIP -ScriptBlock { (Get-WebAppPoolState -Name $args[0]).Value -eq "Stopped" } -argumentlist $env:siteName)
        {
            Invoke-Command -ComputerName $env:serverIP -ScriptBlock { Start-WebAppPool -Name $args[0] } -argumentlist $env:siteName
        }
            
        do
        {
            Invoke-Command -ComputerName $env:serverIP -ScriptBlock { Start-WebAppPool -Name $args[0] } -argumentlist $env:siteName
            Start-Sleep -Seconds 1
        }
        until ( Invoke-Command -ComputerName  $env:serverIP -ScriptBlock { (Get-WebAppPoolState -Name $args[0]).Value -eq "Started" } -argumentlist $env:siteName  )
        ```
5. 建立 Pipeline job
    - 建立 TestPipeline 的 Pipeline 專案
        
        ![7pipeline](https://cloud.githubusercontent.com/assets/3851540/23155472/a5264aca-f84d-11e6-8f89-240551f49d9c.png)
    - 建立使用其他 build job 時需要的參數
    - 撰寫 pipeline script
        - 逐步定義步驟
            1. BuildProject
            2. StopWebSite
            3. Deploy
            4. StartWebSite
        - 給定參數
            
            ![8para](https://cloud.githubusercontent.com/assets/3851540/23155473/a528ffae-f84d-11e6-843f-b1a28608e1ac.png)  
            1. 需要指定參數型態
            2. 指定參數名稱
            3. 指定參數值
        
        ```groovy
        node {
            stage ('BuildProject') {
            		build job: 'TestDotNetProject'
            	}
            stage ('StopWebSite') {
            		build job: 'StopWebSite', parameters: [[$class: 'StringParameterValue', name: 'serverIP', value: params.serverIP],[$class: 'StringParameterValue', name: 'siteName', value: params.siteName]]   
            	}    
            stage ('Deploy') {
            		build job: 'DeployWebSite', parameters: [[$class: 'StringParameterValue', name: 'SourceFolder', value: params.SourceFolder],[$class: 'StringParameterValue', name: 'DestinationFolder', value: params.DestinationFolder]]   
            	}
            stage ('StartWebSite') {
            		build job: 'StartWebSite', parameters: [[$class: 'StringParameterValue', name: 'serverIP', value: params.serverIP],[$class: 'StringParameterValue', name: 'siteName', value: params.siteName]]   
            	}
            }
        ```
## Build
1. 執行 build (給定參數)
    
    ![9buildnow](https://cloud.githubusercontent.com/assets/3851540/23155476/a52ec6fa-f84d-11e6-9c28-6d2fa197ef1f.png)
2. 逐步執行
    
    ![10executebuild](https://cloud.githubusercontent.com/assets/3851540/23155474/a52d0b9e-f84d-11e6-9a3d-239faf895706.png) 


# 參考資料
1. [jenkins 2.0](http://jenkins.readbook.tw/jenkins/jenkins2/index.html)
2. [Jenkins 2.0 初體驗：Pipeline 的使用](http://trunk-studio.com/blog/jenkins-2-preview/)
3. [How to use MsBuild MsDeployPublish to target local file system?](http://stackoverflow.com/questions/9169341/how-to-use-msbuild-msdeploypublish-to-target-local-file-system)
4. [Getting Started with Pipeline](https://jenkins.io/doc/book/pipeline/getting-started/)
5. [Jenkins & Groovy – accessing build parameters](https://rucialk.wordpress.com/2016/03/17/jenkins-groovy-accessing-build-parameters/)