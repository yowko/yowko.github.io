---
title: "將 Jenkins 建立在 Windows Container 上"
date: 2017-08-27T20:17:00+08:00
lastmod: 2018-09-25T20:17:21+08:00
draft: false
tags: ["Docker","Jenkins","Windows Server 2016"]
slug: "jenkins-windows-container"
aliases:
    - /2017/08/jenkins-windows-container.html
---
# 將 Jenkins 建立在 Windows Container 上
公司服務有逐步採用 container 技術的打算，首先第一步就是將 CI Server 給搬進 container，而這個想法老早就有人實際應用了，只是大多應用在 linux 上，對於使用 .Net 技術的團隊幫助有限，因為傳統 .Net 技術的 compiler 需要 .Net Framework 無法完全擺脫 Windows 環境

身為 .NET 開發工程師，就來看看如何將 Windows 環境中的 Jenkins 搬到 container 中吧

## 前置作業

> 以下指令請使用 PowerShell 執行

1.  建立 Jenkins container 使用的資料夾


    *   建立根資料夾

        ```ps1
        New-Item -Type Directory -Name jenkins -Path c:\
        ```

        ![1jenkins](https://user-images.githubusercontent.com/3851540/29749641-81f89afe-8b63-11e7-9c89-e76a508c2401.png)

    *   建立 Jenkins 安裝檔存放資料夾

        ```ps1
        New-Item -Type Directory -Name setup -path C:\jenkins\
        ```

        ![2setup](https://user-images.githubusercontent.com/3851540/29749642-823106f0-8b63-11e7-91f3-d12011fe9dc4.png)

2.  下載 Jenkins

    > 請自行至 [官網](https://jenkins.io/download/) 下載安裝程式，並解壓縮取得 `Jenkins.msi`

3.  將 Jenkins 安裝檔複製至 `setup` 資料夾中

    ![3msi](https://user-images.githubusercontent.com/3851540/29749643-8268fe3e-8b63-11e7-9beb-da5fa23d564d.png)

## 建立 container

1.  建立 container 並直接執行 PowerShell

    > 指定 port mapping 及 volume folder mapping

    ```
    docker run -it -p 8080:8080 --name winjenkins -v c:\jenkins:c:\jenkins microsoft/windowsservercore powershell
    ```

2.  安裝 jenkins

    ```
    msiexec.exe /i C:\jenkins\setup\jenkins.msi /qn
    ```

3.  安裝完成後在 `C:\Program Files (x86)\` 會出現 `jenkins` 資料夾

    > 因為使用靜默安裝，安裝完成不會有提示，安裝大約耗時一兩分鐘

    ![5folder](https://user-images.githubusercontent.com/3851540/29749646-826aa324-8b63-11e7-81ed-239d5b797dcb.png)

## 正常使用

1.  在 container 的 host server 上需要使用 container 內部 ip 來連線

    > windosw container 尚未支援 `localhost` port mapping 至 container 中

    ![6host](https://user-images.githubusercontent.com/3851540/29749645-826a7e94-8b63-11e7-9e1e-7caca4b9aa2b.png)

2.  在其他機器中就直接使用 `host server ip` + `前面 mapping 的 port` 來連線

    ![7outsidehost](https://user-images.githubusercontent.com/3851540/29749648-826b5f08-8b63-11e7-9b15-16a510d2b054.png)

3.  取得 jenkins 驗證 key

    > 在 container 中執行 `type 'C:\Program Files (x86)\Jenkins\secrets\initialAdminPassword'`

    ![8initialpassword](https://user-images.githubusercontent.com/3851540/29749644-826a4258-8b63-11e7-84db-c7f601aa0596.png)

## 心得

測試好幾天才成功，分享一下過程：

1.  前幾天一直在自己的 windows server 2016 上進行測試，後來才發現我的 windows server 2016 上 container 網路一直有問題，但我認為這應該是個案，記得之前在其他 windows server 2016 上並沒有這個問題

2.  在建立 container 時就直接進入 container 安裝 jenkins 是最容易的。我嘗試過先啟動 container 再透過 `docker exec -it winjenkins powershell` 進入 container，發現 container 已經出現 exited，原因還沒有細查

3.  在 host 上執行 `type 'C:\Program Files (x86)\Jenkins\secrets\initialAdminPassword'` 會有權限問題而無法直接取得 `initialAdminPassword`

4.  以上作法個人覺得還有改善空間，包含 image 大小、安裝簡化，但至少驗證在 windows 上安裝 container 是可行的

# 參考資訊
1.  [How do you attach and detach from Docker's process?](https://stackoverflow.com/questions/19688314/how-do-you-attach-and-detach-from-dockers-process)
2.  [Installing JENKINS through Docker File For Windows Container](https://www.assistanz.com/installing-jenkins-through-docker-file-for-windows-container/docker)
3.  [Docker run reference](https://docs.docker.com/engine/reference/run/)
