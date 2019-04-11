---
title: "使用 dockerfile 建立 Windows Container 版 Jenkins"
date: 2017-08-28T00:35:00+08:00
lastmod: 2018-09-25T00:35:11+08:00
draft: false
tags: ["Docker"]
slug: "dockerfile-windows-container-jenkins"
aliases:
    - /2017/08/dockerfile-windows-container-jenkins.html
---
# 使用 dockerfile 建立 Windows Container 版 Jenkins
之前文章 [將 Jenkins 建立在 Windows Container 上](https://blog.yowko.com/2017/08/jenkins-windows-container.html) 分享了如何將 Jenkins 建立在 Windows container 中，而其文末也提到透過文章中介紹的方法來建立 Jenkins container 還有一些缺點待改善，像是 image 的大小及安裝太繁瑣等問題

今天就先來解決安裝太繁瑣的問題 - 透過 dockerfile 來建立 Windows Jenkins container

## 前置作業

> 以下指令請使用 PowerShell 執行

1. 建立 Jenkins container 使用的資料夾

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

4.  在 Jenkins container 資料夾中建立 `dockerfile`

    ```ps1
    New-Item -Type File -Name dockerfile -Path C:\jenkins\
    ```

    ![4dockerfile](https://user-images.githubusercontent.com/3851540/29751971-c8bee3ca-8b87-11e7-9ad6-920b600128ab.png)

5.  加上 dockerfile 內容

    ```dockerfile
    #指定基礎 os image
    FROM microsoft/windowsservercore
    #將 jenkins 安裝檔複製到 container 中
    ADD ./setup c:/jenkins
    #安裝 jenkins
    RUN ["msiexec.exe", "/i", "C:\\jenkins\\jenkins.msi", "/qn"]
    #移除 container 中的安裝檔(讓 image 保持乾淨)
    RUN Powershell.exe -Command remove-item c:/jenkins -Recurse
    #對外使用 8080 port
    EXPOSE 8080  
    #如果有 slave 需加上 50000 port
    EXPOSE 50000
    ```

## 建立 docker image 及 container

1.  編譯 image

    ```
    docker build -t yowko/winjenkins:latest c:\jenkins\
    ```

    *   編譯完成

        ![5buildsuccess](https://user-images.githubusercontent.com/3851540/29751972-c8e7a6c0-8b87-11e7-9648-efc79559139c.png)

    *   確認建立 image

        ![6image](https://user-images.githubusercontent.com/3851540/29751973-c9141e9e-8b87-11e7-972a-ce02af9ad907.png)

2.  建立 container

    ```
    docker run -it --name winjenkins -p 8080:8080 yowko/winjenkins:latest powershell
    ```

    - 這邊透過互動模式執行 `powershell` 讓 container 運行起來，否則 container 還是會自動 stop，進到 container 中後再透過 `ctrl+p, ctrl+q` 退出，讓 container 持續執行

    - 或是執行一個永不停止的指令來讓 container 持續運行 `docker run -d -p 8080:8080 --name winjenkins yowko/winjenkins:latest ping localhost -t`

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

透過 dockerfile 來 build 出 image 比起從無到有手動安裝，看起步驟比較少，只是加上嘗試安裝語法及反覆編譯的時間，如果只用一次，在時間上並不是很划算，但有經常更新版本的需求時，dockerfile 就會省不少事

以前文提到需要改善的部份，今天先針對安裝繁瑣問題做了一些調整，但對於讓 container 持續執行的做法，可能還要再找找其他方式，總覺得不夠聰明

如果懶得自行建立 dockerfile 再 build 成 image，我把做好的 image 上傳到 docker hub 了，可以直接使用 [yowko/winjenkins](https://hub.docker.com/r/yowko/winjenkins/)

# 參考資訊

1.  [將 Jenkins 建立在 Windows Container 上](https://blog.yowko.com/2017/08/jenkins-windows-container.html)
2.  [Installing JENKINS through Docker File For Windows Container](https://www.assistanz.com/installing-jenkins-through-docker-file-for-windows-container/)
3.  [Correct way to detach from a container without stopping it](https://stackoverflow.com/questions/25267372/correct-way-to-detach-from-a-container-without-stopping-it)
4.  [yowko/winjenkins](https://hub.docker.com/r/yowko/winjenkins/)
