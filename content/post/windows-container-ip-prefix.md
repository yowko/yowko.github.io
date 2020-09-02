---
title: "修改 Windows container 的預設 IP"
date: 2017-09-03T23:06:00+08:00
lastmod: 2020-05-02T23:06:20+08:00
draft: false
tags: ["docker","Windows Server 2016"]
slug: "windows-container-ip-prefix"
aliases:
    - /2017/09/windows-container-ip-prefix.html
---
之前文章 [docker 無法連線至特定網段 (172.17.x.x)](https://blog.yowko.com/2017/09/docker-172-17-ip.html) 分享了 docker 的 linux container bridge 會將特定 IP (172.17.*.*) 的 request 導向 container 內部而造成無法存取特定 IP (172.17.*.*) 問題，也介紹如何修改預設分配 IP 規則避免出現 172.17.*.* 的 request 無效

經過 linux container 的折騰，可以預想到 windows container 應該也會有相同問題，就讓我們來看看如何修改 Windows container 分配 IP 的規則吧

## 確認使用的 IP 配發規則

1.  Windows

    > PowerShell 語法：`Get-NetNAT`

    ![1winipam](https://user-images.githubusercontent.com/3851540/30004096-771006d4-90fb-11e7-88ed-1db22c5d9d39.png)

2.  Linux

    > `docker network inspect bridge`

    ![2linuxipam](https://user-images.githubusercontent.com/3851540/30004097-7735f646-90fb-11e7-9f3b-b78fdbb18577.png)

## 修改 Windows container 配合規則

> linux container 的修改方式請參考 [docker 無法連線至特定網段 (172.17.x.x)](https://blog.yowko.com/2017/09/docker-172-17-ip.html)

1. 停止 docker service

    ```ps1
    Stop-Service docker
    ```

2. 刪除所有 container 網路設定

    ```ps1
    Get-ContainerNetwork | Remove-ContainerNetwork -force
    ```

3. 刪除 `NAT` (vSwitch 仍然保留)

    ```ps1
    Get-NetNat | Remove-NetNat
    ```

4. 建立新的 container 網路設定

    ```ps1
    New-ContainerNetwork –name yowkonat –Mode NAT –subnetprefix 192.168.1.0/24
    ```

    ![3newcontainernetwork](https://user-images.githubusercontent.com/3851540/30004098-7755c836-90fb-11e7-8b5a-b28530bd05c9.png)

5. 重新啟動 docker service

    ```ps1
    Start-Service docker
    ```

6. 實際效果

    > 建立新的 container 後，已成功使用前面指定的 `192.168.1.*` ip

    ![4newip](https://user-images.githubusercontent.com/3851540/30004099-775cd05e-90fb-11e7-99ef-89aada998b8b.png)

## 心得

雖然操作只有短短五個步驟，但為了確認正確的指令及步驟，至少重建了 docker 環境不下三十次，花了整整一天的時間呀，主因是一周前我才使用過別的方法達成一樣目的，但才過一周同樣的方法卻無法使用，讓我開始懷疑之前成功是不是只是一場誤會

之前使用的方法是在 windows 環境下，透過修改 `C:\ProgramData\Docker\config\daemon.json` 就可以讓 windows container 使用自定 ip 前綴，但這兩天撰文時發現，windows 下的 docker config 根本不支援 `bip`，使用 `fixed-cidr` 也無效。難道是改版太快嗎？ 或是根本是我記錯 @@"

# 參考資訊

1.  [Docker Engine on Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon?WT.mc_id=DOP-MVP-5002594)
2.  [docker 無法連線至特定網段 (172.17.x.x)](https://blog.yowko.com/2017/09/docker-172-17-ip.html)
3.  [Set up a NAT network](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network?WT.mc_id=DOP-MVP-5002594)
