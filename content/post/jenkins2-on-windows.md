---
title: "Windows OS 安裝 Jenkins 2.0 紀實"
date: 2016-12-30T00:42:34+08:00
lastmod: 2018-09-08T00:42:34+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins2-on-windows"
aliases:
    - /2016/12/windows-install-jenkins-2.html
---
# Windows OS 安裝 Jenkins 2.0 紀實
Jenkins 2.0 主要多了 Pipeline 特性，功能上跟 Workflow plgin 雷同，可以自行定義建置流程，也因為是由 Groovy 語法組成，讓整個建置 task 都可以被版控，而達成 infrastructure as code 目的，於是趁著硬體更新一併進行 Jenkins 2.0 的導入，由於之前使用 Jenkins 1 時沒有留下筆記，結果在架 Jenkins 2.0 時又回憶了好一陣子XD  於是立馬來紀錄囉

## 系統要求
1. 最低要求
    - Java 7
    - 256MB 可用記憶體
    - 1GB+ 可用硬碟空間

2. 建議要求
    - Java 8
    - 1GB+ 可用要求
    - 50GB+ 可用硬碟空間


## 安裝 Jenkins
1. 下載對應 os 的安裝檔
    
    ![1download](https://cloud.githubusercontent.com/assets/3851540/21677006/2efa537c-d373-11e6-9b6b-e48f0428d59a.png)

2. 直接解壓安裝
    
    ![2install](https://cloud.githubusercontent.com/assets/3851540/21677007/2efb506a-d373-11e6-8ade-0580a6846b52.png)

3. 安裝完成
    
    ![3installed](https://cloud.githubusercontent.com/assets/3851540/21677010/2f004f02-d373-11e6-9cf3-da332772c47d.png)

4. 解鎖 Jenkins
    - 安全限制，必需到指定位置 `%ProgramFiles(x86)%\Jenkins\secrets\initialAdminPassword`(安裝目錄下) 取得密碼
        
        ![4unlock](https://cloud.githubusercontent.com/assets/3851540/21677009/2eff12f4-d373-11e6-9202-63b5847b50fa.png)


***如果連網需要透過 proxy 才需要進行下列步驟***
* 無法安裝 plugin
    
    ![5plugin](https://cloud.githubusercontent.com/assets/3851540/21677011/2f1c1066-d373-11e6-8f91-5699f156a5d3.png)

* 設定 proxy
 1. 伺服器
    
    > proxyserver  
 
 2. 連接埠
    
    > proxyport  
 
 3. 使用者名稱
    
    > username  
 
 4. 密碼
    
    > password  
 
 5. 不要代理的主機
    
    > whitelist  
 
 6. Test Url
    
    > 測試 proxy 是否正確的 url

***如果連網需要透過 proxy 才需要進行上述步驟***

## 第一次登入需建立管理者

![7firstadmin](https://cloud.githubusercontent.com/assets/3851540/21715702/08ed0b9c-d441-11e6-932f-3f643694fa9a.png)

- Create First Admin User
    
    ![9addadmin](https://cloud.githubusercontent.com/assets/3851540/21715701/08e9674e-d441-11e6-97db-608b54e3ccb0.png)
    
    - 刪除 `secrets\initialAdminPassword`
    - 刪除 `users\admin\config.xml`
    - 新增 `jenkins.install.InstallUtil.lastExecVersion`
    - 新增 `users\{usernam}\config.xml`

- Continue as admin
    
    ![8asadmin](https://cloud.githubusercontent.com/assets/3851540/21715700/08e764a8-d441-11e6-9d47-ac00750c6a79.png)
    
    - 新增 `jenkins.install.InstallUtil.lastExecVersion`
    - username:admin
    - password:- 使用 `\Jenkins\secrets\initialAdminPassword`(安裝目錄下) 取得密碼

## 安裝 plugin

> - 依實際需求來安裝 e.g. git,msbuild,...etc
> - 其中個人建議應該要安裝的是 `simple theme`，以下就以 `simple theme` 為例紀錄 plugin 的安裝步驟

1. 管理 Jenkins
    
    ![6-1mange](https://cloud.githubusercontent.com/assets/3851540/21677012/2f1d7ef6-d373-11e6-8f4c-9c6ea9aa4d35.png)

2. 管理外掛程式
    
    ![6-2plgin](https://cloud.githubusercontent.com/assets/3851540/21677016/2f22cbe0-d373-11e6-9521-9c6ab067ade2.png)

3. 可用的 --> 搜尋 `simple theme` --> 勾選 --> 安裝
    
    ![6-3filter](https://cloud.githubusercontent.com/assets/3851540/21677013/2f1e22b6-d373-11e6-8a25-745bdac19454.png)

4. 安裝完成
    - 需重啟 Jenkins 
    - http://jenkinsServer/restart 可用來重啟
        
        ![6-4installed](https://cloud.githubusercontent.com/assets/3851540/21677014/2f20497e-d373-11e6-8b73-abc5478a7651.png)

5. 設定 theme
    - 5-1. 管理 Jenkins
        
        ![6-1mange](https://cloud.githubusercontent.com/assets/3851540/21677012/2f1d7ef6-d373-11e6-8f4c-9c6ea9aa4d35.png)
  
    - 5-2. 系統設定
        
        ![6-5system](https://cloud.githubusercontent.com/assets/3851540/21677015/2f22be34-d373-11e6-97a6-c0e6eb95068a.png)
    
    - 5-3. 設定 theme
        - 可以至 [jenkins-material-theme](http://afonsof.com/jenkins-material-theme/) 挑選喜歡的 theme
            
            ![6-6theme](https://cloud.githubusercontent.com/assets/3851540/21677008/2efc4d4e-d373-11e6-911d-9e0984399157.png) 
    
    - 5-4. 效果
        
        ![6-7result](https://cloud.githubusercontent.com/assets/3851540/21677005/2ef9cbe6-d373-11e6-9b6c-cb736a899c2d.png)

設定完是不是比較高級可能見人見智，但至少畫面生動多了

# 參考資料
1. [jenkins 官網](https://jenkins.io)
2. [jenkins-material-theme](http://afonsof.com/jenkins-material-theme/)