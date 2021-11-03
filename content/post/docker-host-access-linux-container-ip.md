---
title: "如何從 Winows docker host 透過 linux container ip 使用服務"
date: 2017-09-14T23:16:00+08:00
lastmod: 2021-11-03T23:16:03+08:00
draft: false
tags: ["Docker","Windows","Linux"]
slug: "docker-host-access-linux-container-ip"
aliases:
    - /2017/09/docker-host-access-linux-container-ip.html
---
## 如何從 Winows docker host 透過 linux container ip 使用服務

前幾天為了重現前幾天在 production 環境遇到的 issue，嘗試在 Windows Server 2016 透過 docker for windows 建立 linux 版 redis，因為需要模擬 sentinel failover 所以需要先測試看看 redis 各個 instance 是否有正確建立，想不到從 host 上連線就失敗了XD

這應該是滿常見的需求：開發階段從本機測試服務是否正常運作，就來看看該如何設定吧

## 重現問題

> host 無法存取 container ip

1. 取得 container ip

    ```cmd
    docker inspect container_name_or_container_id | findstr '"IPAddress"'
    ```

    ![1getip](https://user-images.githubusercontent.com/3851540/30390143-ae1a625c-98e7-11e7-9f6f-5691ef81b5b7.png)

2. 無法存取 container ip

    ![2containeripfail](https://user-images.githubusercontent.com/3851540/30390145-ae1bdcc2-98e7-11e7-8b45-7f21970543f3.png)

3. 透過 host ip 正常

    ![4localhostOK](https://user-images.githubusercontent.com/3851540/30390146-ae25b10c-98e7-11e7-9424-2819aa7fd801.png)

4. 透過 localhost 也正常

    ![3hostipok](https://user-images.githubusercontent.com/3851540/30390141-ae153674-98e7-11e7-8d70-7b1a219ea53b.png)

## 如何解決

1. 手動加上一筆 route 讓 host 可以 access 至 container ip
    * 確認 container 使用的 router ip

        ![5routerip](https://user-images.githubusercontent.com/3851540/30390142-ae195330-98e7-11e7-89a2-db94a3906782.png)

        * 新版的 MobyLinuxVM 已經看不到 IP 了

            ![6NOIP](https://user-images.githubusercontent.com/3851540/30390151-b06ccd10-98e7-11e7-8104-3d0209b0b08d.png)

    * 加上 route 紀錄

        ```bash
        route /P add 172.0.0.0 MASK 255.0.0.0 10.0.75.2
        ```

    * 檢查是否有加入成功

        ```bash
        route print
        ```

        ![7routeadded](https://user-images.githubusercontent.com/3851540/30390147-ae5073c4-98e7-11e7-9050-1ea3786bc863.png)

2. 已可正常存取 container ip 服務

    ![8accessok](https://user-images.githubusercontent.com/3851540/30390144-ae1ae934-98e7-11e7-83f7-ed95f6d8022b.png)

3. 如果依上述操作仍無法存取 container 服務，請重啟 docker service 即可

## 心得

Windows 上的 docker 雖然已成熟許多，但仍有許多改善的空間，使用起來還是有很多眉眉角角需要了解，但我相信依微軟過去風格，應該要不了多少，就會改良至無腦使用的階段，到時相信使用門檻會大幅降低的，不然依目前使用經驗，會用的人都需要不少軟硬體相關知識才有辦法解決遭遇到的狀況，實在不親民呀

## 參考資訊

1. [How to access containers by internal IPs 172.x.x.x](https://github.com/docker/for-win/issues/221)
2. [Docker for Windows](https://cianallner.com/docker-for-windows/)
