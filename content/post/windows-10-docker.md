---
title: "Windows 10 如何啟用 docker 功能"
date: 2017-05-08T01:08:00+08:00
lastmod: 2018-09-18T01:08:03+08:00
draft: false
tags: ["Docker","Windows 10"]
slug: "windows-10-docker"
aliases:
    - /2017/05/windows-10-docker.html
---
# Windows 10 如何啟用 docker 功能
Windows 10 雖然不是原生支援 Windows Server container(透過 Hyper-V 支援)，執行速度較慢，但 Microsoft 在 Windows 10 上將 windows container 與 Linux container 包裝得非常方便。只要滑鼠動幾下就可以在不需人為介入下使用 Hyper-V 建立 Windows Server container 及 Linux container ，使用者體驗很好，反而在 Windows Server 2016 上還需要手動使用 Hyper-V 安裝 Linux 後才能建立 Linux container (2017/5 當下現況，Microsoft 已經對外公布會解決這個問題，只是沒說什麼時候而已XD)，複雜度遠遠超過 Windows 10，加上本來對 Linux 就不熟悉，要完全搞定這些環境是個非常大的挑戰，在兩方優劣比較下，決定暫時犧牲效能先求可以正常使用就好，畢竟重點是 container 而不是 os，立馬來看看該如何讓 Windows 10 可以使用 docker 吧

## 必要條件

1.  Windows 10 Anniversary Edition(年度版) 或是 Creators Update (Professional or Enterprise).

    > 如果沒有的話可以至微軟官方 [下載 Windows 10 光碟映像 (ISO 檔案)](https://www.microsoft.com/zh-tw/software-download/windows10ISO)

2. 確認 Windows 10 的 OS 組建版本需要 14393.222 以上

    > 開啟 commnad prompt 或 Windows PowerShell 執行 `winver.exe`

    ![1build](https://cloud.githubusercontent.com/assets/3851540/25774324/92e2ce90-32bf-11e7-8776-7640da31c656.png)

## 啟用容器功能

1.  啟用 Hyper-V
    *   使用管理者權限開啟 Windows PowerShell
    *   執行 `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All`

        ![2installhyperv](https://cloud.githubusercontent.com/assets/3851540/25774326/930fb6d0-32bf-11e7-8b53-08b1e5f6b087.png)

    *   提示重新啟動電腦(可以等 container 功能啟用後再重新啟動即可)

        ![3restart](https://cloud.githubusercontent.com/assets/3851540/25774325/930f7e68-32bf-11e7-835f-99f984063e1d.png)

2.  啟用 container
    *   使用管理者權限開啟 Windows PowerShell
    *   執行 `Enable-WindowsOptionalFeature -Online -FeatureName containers -All`

        ![4installcontainer](https://cloud.githubusercontent.com/assets/3851540/25774327/931203ae-32bf-11e7-9015-be888e1bb063.png)

    *   重新啟動電腦

        ![5restart](https://cloud.githubusercontent.com/assets/3851540/25774329/93377206-32bf-11e7-9da7-d4259bbde6a5.png)

        *   或是執行 `Restart-Computer -Force`

## 安裝 Docker for Windows

1. 下載 [Docker for Windows](https://download.docker.com/win/stable/InstallDocker.msi)

2. 執行 `InstallDocker.msi` 安裝 Docker for Windows

    ![6installdocker](https://cloud.githubusercontent.com/assets/3851540/25774331/9337c47c-32bf-11e7-8f3c-6f687b1d3114.png)

    *   安裝完成後請重新開機

3.  確認安裝狀態

    > 開啟 commnad prompt 或是 Windows PowerShell ，執行 `docker info`

    ![7dockerinfo](https://cloud.githubusercontent.com/assets/3851540/25774328/93371310-32bf-11e7-940f-a9ea4d94eedf.png)

## 建立 container

*  系統圖示列可在 Windows conatiner 與 Linux container 間自行切換

    > 切換需要一點時間，畢竟背景會默默啟動 Hyper-V

    *   to Linux container


        ![8towindowscontainer](https://cloud.githubusercontent.com/assets/3851540/25774332/9339bc00-32bf-11e7-9412-6335c6eed595.png)

    *   to windows container

        ![9toLinuxcontainer](https://cloud.githubusercontent.com/assets/3851540/25774330/93378304-32bf-11e7-8866-b82514a61a11.png)

*  可以透過 commnad prompt 或是 Windows PoerShell 來切換

    *   commnad prompt

        ```
        "C:\Program Files\Docker\Docker\DockerCli.exe" -SwitchDaemon
        ```

    *   Windows PoerShell

        ```ps1
        & 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon`
        ```
        
* 如果使用錯誤 os 的 image 會有提示

    *   Linux use windows image

        ![13Linuxusewindows](https://cloud.githubusercontent.com/assets/3851540/25774323/92e1f9f2-32bf-11e7-95be-b6532dcba590.png)

    *   windows use Linux image

        ![11windowsuseLinux](https://cloud.githubusercontent.com/assets/3851540/25774334/9360b4b8-32bf-11e7-8a1d-cf11303d0e6c.png)

1. Linux container

    > `docker run -it --name centos centos`

    ![10Linuxok](https://cloud.githubusercontent.com/assets/3851540/25774333/933a440e-32bf-11e7-8b85-8fee469b0c64.png)

2.  Windows Server container

    > `docker run -it microsoft/nanoserver cmd`

    ![12windowerok](https://cloud.githubusercontent.com/assets/3851540/25774335/9362c42e-32bf-11e7-8a03-8cb4f7f109ad.png)

## 心得

Windows 因為系統本身大小的問題，在取得 base image 或是建立 container 時執行速度都比 Linux 慢上不少，除此之外整體使用體驗與 Linux 環境幾乎相同，不由得佩服 Microsoft 的決心，Windows container 非常值得期待，想必會在 .Net 開發界有不小的衝擊

# 參考資訊
1.  [Windows Containers on Windows 10](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10?WT.mc_id=DOP-MVP-5002594)
2.  [Windows 10 上的 Windows 容器](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/quick-start/quick-start-windows-10?WT.mc_id=DOP-MVP-5002594)
3.  [如何在 Windows 10 同時安裝與執行 Windows 與 Linux 容器 (Docker)](http://blog.miniasp.com/post/2016/11/22/Run-Linux-and-Windows-Containers-on-Windows-10.aspx)
