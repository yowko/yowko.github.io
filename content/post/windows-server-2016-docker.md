---
title: "如何在 Windows Server 2016 安裝 Docker"
date: 2017-05-04T01:00:00+08:00
lastmod: 2018-09-18T01:00:04+08:00
draft: false
tags: ["Docker","Windows Server 2016"]
slug: "windows-server-2016-docker"
aliases:
    - /2017/05/windows-server-2016-docker.html
---
# 如何在 Windows Server 2016 安裝 Docker
Windows Server 2016 是第一個原生支援 Docker 的 Windows 作業系統，雖然 windows container 的 image 數量遠遠比不上 linux container，但 MicroSoft 自家出品的 windows 原生相關產品與工具還是可以透過 Windows Server 2016 上的 Docker 來執行，首先就來看看該如何設定

## 使用 PowerShell 安裝 Docker

1.  使用管理者權限開啟 Windows PowerShell
2.  安裝 `Docker-Microsoft PackageManagement Provider`

    ```ps1
    Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
    ```

    *   提示需要 NuGet

        ![1nuget](https://cloud.githubusercontent.com/assets/3851540/25586234/0bcfaf3c-2ed1-11e7-95aa-a3b3717396d0.png)

    *   下載並安裝 NuGet


        ![2download](https://cloud.githubusercontent.com/assets/3851540/25586235/0beba4f8-2ed1-11e7-892c-fba077d69bb4.png)

3.  安裝 Docker

    ```ps1
    Install-Package -Name docker -ProviderName DockerMsftProvider
    ```

    *   安全性提示 --> 全部皆是

        ![3security](https://cloud.githubusercontent.com/assets/3851540/25586239/0bf27de6-2ed1-11e7-829e-ff988a628c77.png)

    *   安裝完成提示需重新開機

        ![4restart](https://cloud.githubusercontent.com/assets/3851540/25586237/0bec48ae-2ed1-11e7-8cbb-0059d56cc014.png)

4.  重新開機

    ```ps1
    Restart-Computer -Force
    ```

## 安裝 Windows Updates

1. 以管理員開啟 command prompt 或是 Windows PowerShell 並檢查更新

    ```ps1
    sconfig
    ```

    ![5sconfig](https://cloud.githubusercontent.com/assets/3851540/25586236/0bebe972-2ed1-11e7-94b5-5ef795dca366.png)

2.  下載並安裝更新

    > 選 `6`

    ![6sconfig6](https://cloud.githubusercontent.com/assets/3851540/25586238/0bef6250-2ed1-11e7-97c9-1cdf3fa2c3e2.png)

3.  搜尋所有更新

    > `A`

    ![7searchall](https://cloud.githubusercontent.com/assets/3851540/25586240/0bf6eb74-2ed1-11e7-9509-608543d6f0d1.png)

4.  下載所有更新

    ![8downloadall](https://cloud.githubusercontent.com/assets/3851540/25586241/0c11afd6-2ed1-11e7-8da4-dc0f9ec66915.png)

5.  安裝結果

    ![9installresult](https://cloud.githubusercontent.com/assets/3851540/25586242/0c121f16-2ed1-11e7-929e-587ee967deec.png)

## 啟動 Docker service

1.  開啟 Services.msc

    ![10services](https://cloud.githubusercontent.com/assets/3851540/25586243/0c12d500-2ed1-11e7-8a79-0239c420245f.png)

2.  啟動 docker
    *   docker service --> 按右鍵 --> 啟動

        ![11startup](https://cloud.githubusercontent.com/assets/3851540/25586244/0c1661ac-2ed1-11e7-83a9-1c450dbdd300.png)

3.  確認啟動狀態

    > docker info

    *   未啟動

        ![12deactive](https://cloud.githubusercontent.com/assets/3851540/25586245/0c1b8ef2-2ed1-11e7-996b-3afab1c88aa0.png)

    *   正常啟動

         ![13active](https://cloud.githubusercontent.com/assets/3851540/25586233/0ba47fd8-2ed1-11e7-9414-ff73207ccfdc.png)

# 參考資訊
1.  [Windows Containers Quick Start](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/?WT.mc_id=DOP-MVP-5002594)
2.  [Windows Containers on Windows Server](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-server?WT.mc_id=DOP-MVP-5002594)
