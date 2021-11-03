---
title: "如何修改 Jenkins 2 預設使用的 port 以及在同一 Windows Server 安裝多個 Jenkins"
date: 2017-02-08T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps"]
slug: "change-jenkins-port-multiple-jenkins"
aliases:
    - /2017/02/change-jenkins2-port-and-multiple-jenkins.html
---
## 如何修改 Jenkins 2 預設使用的 port 以及在同一 Windows Server 安裝多個 Jenkins

在某些情境下(e.g. slave、專案實體隔離)，需要在同一台 Server 上安裝多個 Jenkins，另外可能因為環境問題需要使用非 8080 port ，就來看看可以怎麼做吧

## 修改 Jenkins port

1. 開啟 Jenkins 安裝資料夾

    > 預設為 `C:\Program Files (x86)\jenkins`
2. 修改 jenkins.xml

    ```xml
    --httpPort=8080
    ```

    - 改為自訂 port

        ![1chnageport](https://cloud.githubusercontent.com/assets/3851540/21677199/09a07330-d374-11e6-9636-5476426ae68f.png)

3. 重啟 jenkins 服務
    - `Jenkins` 右鍵 --> Restart

        ![2restart](https://cloud.githubusercontent.com/assets/3851540/21677198/097a6492-d374-11e6-8036-6e3054091449.png)

## URL 命令

```cmd
http://[jenkins-server]/[command]
```

1. `exit` 關閉 Jenkins
2. `restart` 重啟 Jnekins
3. `reload` 重新載入設定

## 如何安裝多個 Jenkins 2

1. 安裝 Jenkins 2
    - 單一安裝可以參考 [Windows OS 安裝 Jenkins 2.0 紀實](/windows-install-jenkins-2)

2. 複製既有 Jenkins 2
    - 複製整個 Jenkins 資料夾至新位置

3. 修改使用 port
    - Jenkins 資料夾 下 jenkins.xml
     改為自訂 port

        ![1chnageport](https://cloud.githubusercontent.com/assets/3851540/21677199/09a07330-d374-11e6-9636-5476426ae68f.png)
4. 修改服務 id 及 name
    - <span style="color:red">這個設定會直接影響需要重啟 Jenkins 功能</span>
    - `id` 需與下個步驟 Winodws Service 名稱相同

5. 註冊服務

    ```cmd
    sc.exe create "{ServiceName}" start= auto binPath= "{Jenkins Folder}\jenkins.exe" DisplayName= "{ServiceDisplayName}"
    ```

    - e.g. `sc.exe create "Jenkins2" start= auto binPath= "D:\Jenkins-New\jenkins.exe" DisplayName= "Jenkins 2"`
    - 刪除 `sc delete {ServiceName}`

        ![3createservice](https://cloud.githubusercontent.com/assets/3851540/21677200/09bfaafc-d374-11e6-8b5f-ce2ba1621268.png)

## 心得

1. 再次執行 Windows 的 Jenkins 安裝檔會出現下列問題
    - 預設使用 "Jenkins" Service，新的安裝會把 service binpath 直接改為新的安裝位置
    - 也會移除原安裝資料夾下的必備檔案 (e.g. jenkins.exe jenkins.war jenkins.xml... )
2. 感覺上直接複製不是個好方法，但至少可以達成目的
3. 重新開機後，新建的 Jenkins 沒有自動啟動，手動啟動也出現 `Error 1607`, 請檢查 `jenkins.war` 是否還存在(可以直接從 官網 下載 or 從舊資料夾複製)

    > 可以考慮把 Jenkins 資料夾版控，避免其他檔案也被誤刪了

## 參考資料

1. [Installing Jenkins as a Windows service](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+as+a+Windows+service)
2. [Administering Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/Administering+Jenkins)
