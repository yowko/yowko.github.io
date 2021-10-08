---
title: "Widnows 環境中修改 Docker image 的儲存位置"
date: 2017-08-18T00:06:00+08:00
lastmod: 2021-10-08T00:06:33+08:00
draft: false
tags: ["docker","Windows 10","Windows Server 2016"]
slug: "windows-docker-image-location"
aliases:
    - /2017/08/windows-docker-image-location.html
---
## Widnows 環境中修改 Docker image 的儲存位置

同事在 pull asp-net image 時，發現系統磁碟空間不足，無法取得 image，所以想修改 image 的預設儲存位置，讓 image 可以不必佔用系統磁碟槽

這樣一來也可以讓 image 儲存至 NAS 上，讓其他機器也可以共用這些 image，有效降低網路頻寬跟時間

## Windows Server 2016 與 Windows 10 上的 Windows Container

透過 `docker info` 指令可以看到 docker 相關設定

![1dockerinfo](https://user-images.githubusercontent.com/3851540/29421728-85f3ab16-83a8-11e7-94a9-33e3411baf0c.png)

* 預設 Docker 使用路徑

    > `C:\ProgramData\dcoker`

* 預設 image 儲存資料夾

    > `windowsfilter`

* 修改儲存路徑

    1. 在 `C:\ProgramData\Docker\config` 新增 `daemon.json`
    2. 修改 `daemon.json` 指定儲存路徑 `{"graph": "C:\\Docker"}`
    3. 重新啟動 docker 服務
        * 使用 powershell 執行指令

            > `restart-service *docker*`

    4. 修改完成

        ![2chnaged](https://user-images.githubusercontent.com/3851540/29421726-85ee1930-83a8-11e7-8a92-682d3dc341a0.png)

## Windows 10 上的 Linux Container

Windows 10 上的 Linux container 是透過 hyper-v 的虛擬化技術建立的，因此 linux 相關內容都存在 VHD 中

* 開啟 docker daemon --> setting

    ![3dockerdaemon](https://user-images.githubusercontent.com/3851540/29421725-85ecc01c-83a8-11e7-9282-e7515b198c1f.png)

* Advanced --> Images and volumes VHD location

    ![4imagelocation](https://user-images.githubusercontent.com/3851540/29421727-85ef12c2-83a8-11e7-8f37-c4be16ca7916.png)

## 心得

剛好這次遇到問題是 Windows 環境，我猜 linux 環境上應該也會有類似的需求，待我的 linux 環境搞定後再來研究看看該如何設定

## 參考資訊

1. [How to change docker images and containers location with Windows Containers?](https://social.technet.microsoft.com/Forums/windowsserver/en-US/4ac564e2-ad6d-4d32-8cb4-7fea481738a4/how-to-change-docker-images-and-containers-location-with-windows-containers?forum=ws2016)
2. [Restart Docker service from command line](https://forums.docker.com/t/restart-docker-service-from-command-line/27331)
3. [Change Docker native images location on Windows 10 Pro](https://stackoverflow.com/questions/40465979/change-docker-native-images-location-on-windows-10-pro)
