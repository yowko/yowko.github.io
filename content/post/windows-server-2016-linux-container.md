---
title: "在 Windows Server 2016 上使用 Linux Container"
date: 2017-09-01T01:40:00+08:00
lastmod: 2018-09-25T01:40:59+08:00
draft: false
tags: ["docker","Windows Server 2016"]
slug: "windows-server-2016-linux-container"
aliases:
    - /2017/09/windows-server-2016-linux-container.html
---
# 在 Windows Server 2016 上使用 Linux Container
想必大家對於 Windows 10 透過簡易的 GUI 就可達到快速切換 Windows container 與 Linux container 的功能非常有印象(詳細內容可以參考 [Windows 10 如何啟用 docker 功能](https://blog.yowko.com/2017/05/windows-10-docker.html))，原本在 Windows Server 2016 上並沒有提供這樣的工具，這幾天經同事指點：新版的 Docker for Windows 已經支援 Windows Server 2016 可以快速轉換 Windows container 與 Linux container

在同事的催促下，就來紀錄一下該如何安裝與使用吧

## 安裝 `Docker for Windows`

1.  下載 `Docker for Windows`

    ![1download](https://user-images.githubusercontent.com/3851540/29936634-832f318a-8eb5-11e7-9444-fc32d767e153.png)

    > 下載位置：[Docker for Windows Installer.exe](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

2.  安裝 `Docker for Windows`

    ![2install1](https://user-images.githubusercontent.com/3851540/29936633-832e7768-8eb5-11e7-8a65-b02b4834549b.png)

    ![3install2](https://user-images.githubusercontent.com/3851540/29936632-83274538-8eb5-11e7-887c-d174d4646251.png)

    ![4installed](https://user-images.githubusercontent.com/3851540/29936635-8330400c-8eb5-11e7-9033-f2470684d793.png)

## 安裝 Hyper-V

Windows 環境並不支援 Linux 相關 api，為了讓 Windwos Server 2016 可以直接使用 Linux container 需要透過 Hyper-V 建立 Linux 環境來 host Linux container

*   方法 一：透過 Docker for Windows 啟動

    > 安裝完成 `Docker for Windows` 後會直接啟動，預設使用 Linux container 就會直接提示安裝，按下 `OK` 就會安裝 Hyper-V 並重啟

    ![5requirehyperv](https://user-images.githubusercontent.com/3851540/29936637-83580506-8eb5-11e7-9864-a0b9d8e64440.png)

*   方法 二：自行安裝

    > 詳細安裝方式請參考 [如何在 Windows Server 2016 上安裝 Hyper-V](https://blog.yowko.com/2017/05/windows-server-2016-hyper-v.html)

## 安裝完成

1.  具備在 Windows Container 與 Linux Container 間快速切換的功能

    ![6linux](https://user-images.githubusercontent.com/3851540/29936636-8357e274-8eb5-11e7-8642-cf4b874cf403.png)

    ![7windows](https://user-images.githubusercontent.com/3851540/29936641-845849e8-8eb5-11e7-93f1-664f96df1901.png)

2.  使用 Linux Container 時會透過 Hyper-V 自動載入 `MobyLinuxVM` 的 Linux 環境供 Linux container 執行

    ![8moby](https://user-images.githubusercontent.com/3851540/29936638-8362aa38-8eb5-11e7-83e7-71dcd85d560f.png)

## 心得

透過 `docker for windows` 可以讓 Windows Server 2016 也能擁有像是 Windows 10 一樣橫跨 Windows container 與 Linux container 的優點，快速切換的功能讓使用不同 OS container 時更加節省時間及有效率，非常感謝同事指教

另外補充一點：Windows Server 2016 所使用的 windows container 是專為 windows 設計的架構，os 層 api 都是全新設計，而 Windosw 10 上的 windows container 則是透過 Hyper-V 模擬出來的，效能上不如 Windows Server 2016 上原生的 windows container 好。

以下提供簡易辨別 docker 是否運行於 Hyper-V 的方式：

> 使用 `docker info` 語法檢視 windows conatiner 的 isolation 類型

1.  process - 直接使用 windows container

    ![9process](https://user-images.githubusercontent.com/3851540/29936639-83672450-8eb5-11e7-9200-d368cb36ddb1.png)

2.  hyperv - 使用 Hyper-V container

    > 這邊要注意，Hyper-V container 也可能出現在 Windows Server 2016 上，目的是達成更安全的隔離效果

    ![10hyperv](https://user-images.githubusercontent.com/3851540/29936640-839f5c26-8eb5-11e7-8907-e18f13c0cc91.png)

# 參考資訊

1.  [Install Docker for Windows](https://docs.docker.com/docker-for-windows/install/)
2.  [Windows 10 如何啟用 docker 功能](https://blog.yowko.com/2017/05/windows-10-docker.html)
3.  [如何在 Windows Server 2016 上安裝 Hyper-V](https://blog.yowko.com/2017/05/windows-server-2016-hyper-v.html)
