---
title: "Windows Container 使用固定 IP"
date: 2017-11-16T00:47:00+08:00
lastmod: 2020-09-01T00:47:32+08:00
draft: false
tags: ["Docker","Windows 10","Windows Server 2016"]
slug: "windows-container-static-ip"
aliases:
    - /2017/11/windows-container-static-ip.html
---
# Windows Container 使用固定 IP
有時候服務主機本身沒有綁定 FQDN，只透過 IP 來連線存取，這個時候動態分配 ip 就會造成其他服務需要調整連線資訊，相當不方便，所以會利用指定固定 ip 的方式 來暫時解決問題，這個狀況也有可能發生在 container 環境，所以今天就來紀錄該如何為 Windows Container 設定使用固定 ip

## 建立 docker network

> 這是建立 container 時指定 ip 的預先設定

*   格式

    ```
    docker network create -d nat --subnet={subnet ip}/24 --gateway={gatway ip} {network name}
    ```

    *   其中 `-d` 後面接的是 `網路驅動程式類型`，我只用過 `nat` 及 `transparent`，以這兩者而言：`nat` 可以指定 ip 也可以指定 port 對應，但 `transparent` 就只能指定 ip，無法指定 port 對應

*   範例

    ```
    docker network create -d nat --subnet=172.24.1.0/24 --gateway=172.24.1.1 YowkoNat
    ```

    ![1createnetwork](https://user-images.githubusercontent.com/3851540/32848239-ea7cd8d2-ca66-11e7-87b3-069c3a906461.png)

## 啟動 Conatiner 指定 ip

*   格式

    ```
    docker run -d --network={network name} --ip {ip} {image name}
    ```

*   範例

    ```
    docker run -d --network=YowkoNat --ip 172.24.1.2 microsoft/windowsservercore
    ```

    ![2specific](https://user-images.githubusercontent.com/3851540/32848243-eabb941e-ca66-11e7-8f99-44e39e0bcddc.png)

## 心得

不知道是服務還不穩定還是服務本身有 bug，在我完成一次設定後，後續要再重新建立 network 時一直出現 HNS 錯誤，找了些資料始終沒有解決問題，但如果類型指定的是 `transparent` 則都正常，不知道是網路層還是 docker 本身的問題，改天有請教到專家再紀錄分享吧

# 參考資訊

1.  [How To Assign A Static IP Address To A Windows Container](https://www.ntweekly.com/?p=14891)
2.  [Windows 容器的網路功能](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/manage-containers/container-networking?WT.mc_id=DOP-MVP-5002594)
