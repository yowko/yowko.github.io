---
title: "修改 Windows 上 Linux Container (MobyLinuxVM) 的 bip 設定"
date: 2017-09-04T00:51:00+08:00
lastmod: 2021-10-08T00:51:56+08:00
draft: false
tags: ["Docker","Linux","Windows Server 2016"]
slug: "windows-linux-container-mobylinuxvm-bip"
aliases:
    - /2017/09/windows-linux-container-mobylinuxvm-bip.html
---
## 修改 Windows 上 Linux Container (MobyLinuxVM) 的 bip 設定

最近幾篇筆記都是跟 ip 設定有關，主要是因為 docker 預設分配 container 使用的 ip 區段與公司測試環境衝突，造成 docker host 與 container 無法存取測試環境的 server (詳細原因請參考 [docker 無法連線至特定網段 (172.17.x.x)](/2017/09/docker-172-17-ip.html))

之前分別介紹過 linux container 與 windows container 的設定，今天就來分享一下 Windows 上的 Linux container 該如何設定

## 前提設定

> 有兩篇筆記是必需先看過的

1. Linux 上修改 container 預設 IP 設定

    > [docker 無法連線至特定網段 (172.17.x.x)](/2017/09/docker-172-17-ip.html)

2. 使用 docker for windows 來快連切換 linux container 與 windows container

    > [在 Windows Server 2016 上使用 Linux container](/2017/09/windows-server-2016-linux-container.html)

## 如何設定 Windows 上的 linux container

經過上述兩篇文章，相信對於在 linux 環境是透過修改 `/etc/docker/daemon.json` 與如何透過 docker for windows 在 windows 使用 linux container 有些概念，原則上我們就是要去修改 Windows 上的 linux 基礎環境設定，其中 windows 上的 linux 是透過 Hyper-V 建立出來的虛擬環境，如果這個虛擬環境 - MobyLinuxVM 行為跟一般 linux 相同也就沒事但難處就是這個虛擬環境 - MobyLinuxVM 已經被 docker 封裝過，我們無法直接存取其中的資料內容，當然也就修改不到 `/etc/docker/daemon.json`，就來看看該如何調整吧

1. 開啟 docker for windows 的設定

    > docker for windows 圖示按右鍵--> Settings...

    ![1setting](https://user-images.githubusercontent.com/3851540/30004922-1ba4a408-910a-11e7-9d29-316bd128b3d5.png)

2. 進入 Daemon 選單

    ![2daemon](https://user-images.githubusercontent.com/3851540/30004923-1bd09b08-910a-11e7-8ada-f96c0ba40b3b.png)

3. 切換至 `Advanced` 模式

    > 從 `Basic` 切換至 `Advanced` 就可以編輯 config 內容，相關參數就是 `daemon.json` 的用法，詳細內容可以參考官方說明 [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)

    ![3ADVANCED](https://user-images.githubusercontent.com/3851540/30004924-1be7ce22-910a-11e7-8e2e-84b82f86930e.png)

4. 加上 `bip` 設定

    * 具有基本格式檢查

        ![4errorcheck](https://user-images.githubusercontent.com/3851540/30004925-1beb7a5e-910a-11e7-8bba-3df3e8745f42.png)

    * 填寫完成後按下 `Apply`

        ![5apply](https://user-images.githubusercontent.com/3851540/30004926-1bef1bdc-910a-11e7-8f5b-75071e2154cc.png)

    * 會自動重啟 docker service 以套用設定

        ![6restarting](https://user-images.githubusercontent.com/3851540/30004927-1bf396ee-910a-11e7-89cc-771ae0404c37.png)

5. 修改成功

    > container 內部 ip 就會依指定的範圍來分配了

    ![7result](https://user-images.githubusercontent.com/3851540/30004928-1c786586-910a-11e7-8e50-07647aea61d8.png)

## 心得

修改內容很少，但我跟同事查了好多資料一直找不到可用的解法，偶然間才發現的設定，所以動手紀錄一下以資留念

## 參考資訊

1. [docker 無法連線至特定網段 (172.17.x.x)](/2017/09/docker-172-17-ip.html)
2. [在 Windows Server 2016 上使用 Linux container](/2017/09/windows-server-2016-linux-container.html)
3. [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)
