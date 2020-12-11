---
title: "從 mac 使用 Container Ip 直連"
date: 2019-12-22T22:30:00+08:00
lastmod: 2020-12-11T22:30:31+08:00
draft: false
tags: ["macOS","Tools","Helm","Kubernetes","Docker"]
slug: "mac-access-container-ip-host"
---

## 從 mac 使用 Container Ip 直連

大約一年前開發環境從 Windows 轉換至 mac 後，開發上最痛苦的就是沒有 Visual Studio 可以用，痛苦指數第二高的我個人認為是 Docker for Mac 沒有 docker0 網卡 (這是已知的限制：[Networking features in Docker Desktop for Mac - Known limitations, use cases, and workarounds](https://docs.docker.com/docker-for-mac/networking/#known-limitations-use-cases-and-workarounds) )，雖然很多事還是有 workaround 也可以透過許多技巧避開，但我在意的是這些 workaround 與技巧讓 Docker for Mac 的行為與實際 production 環境可能不同

之前有網友在看了 [使用 docker 建立 Redis Cluster - 更新版](/redis-cluster-docker/) 後詢問有沒有辦法讓 redis cluster 可以從外部直連，後來同事也遇到類似問題，心想如果是 linux 或是 Windows 只要多個 route rule，大概一分鐘可以搞定，有卡關的想必跟我一樣都是陷入 docker for mac 天生限制的泥淖中，所以就興起了嘗試看看有沒有辦法突破 Docker for Mac 沒有 docker0 網卡 的這樣限制，讓開發人員可以直接在 mac host 上透過 Container Ip 直接存取 Container 上的服務而不是使用 port mapping 的方式

趁著假日空閒時間終於試出點成果，紀錄一下辛苦的過程(好幾次都差點放棄了)

## 基本環境說明

1. macOS Catalina 10.15.1
2. Docker Desktop for Mac community 2.1.0.5(40692)
3. Kubernetes v1.14.8
4. Helm 2.16.1
5. Tunnelblick 3.8.1 (build 5400)

## 設定方式

核心概念是在 Docker for Mac 的虛擬機中建立 OpenVPN Server，然後從 mac 使用 VPN 連進去，整個流程可以參考 [MAC DOCKER NETWORK TUNNEL](http://www.scispike.com/blog/mac-docker-network-tunnel/) 的圖示

![flow](http://www.scispike.com/wordpress/wp-content/uploads/2016/08/dockerNetwork.png)

>圖片來： [MAC DOCKER NETWORK TUNNEL](http://www.scispike.com/blog/mac-docker-network-tunnel/)

1. 前置作業

    - 啟動任一 container

        > 這邊以 redis 為例

        ```bash
        docker run -d --rm --name redisformac redis
        ```

    - 取得 redis container ip

        > 這個 ip 記得保留，後面會用到

        ```bash
        docker inspect `docker ps | grep redisformac | awk '{print $1}'` | grep '"IPAddress"'
        ```

    - 確認連不到

        ![1pingcontainer](https://user-images.githubusercontent.com/3851540/71323783-31997100-2512-11ea-9b85-cab236308693.png)

        ![2rdmfail](https://user-images.githubusercontent.com/3851540/71323784-32320780-2512-11ea-8afc-e462f117682f.png)

2. 啟用 Docker for Mac 的 Kubernetes 功能

    > 詳細內容請參考之前筆記 [在Docker for Mac 上啟用Kubernetes](/docker-for-mac-kubernetes/)

3. 安裝 Helm

    > 詳細內容請參考之前筆記 [在 Docker for Mac 啟用 Kubernetes 後安裝 Helm](/docker-mac-kubernetes-helm/)

4. 下載 [Development Toolkit for Kubernetes on Docker for Mac](https://github.com/pengsrc/docker-for-mac-kubernetes-devkit)

    > 雖然說明文件中有提到可以使用 docker-compose 執行 (甚至執行方式更簡單)，但我一直沒有成功

    ```bash
    git clone https://github.com/pengsrc/docker-for-mac-kubernetes-devkit.git
    ```

5. 調整或建立 volume 用資料夾 (請依需求調整)

    > 依 `docker-for-mac-kubernetes-devkit/values.yaml` 中的 dirPaths 設定建立指定資料夾

    ```bash
    mkdir /tmp/docker-for-mac-openvpn && mkdir /tmp/openvpn && /tmp/openvpn/configs
    ```

6. 將 `docker-for-mac-openvpn/setup.sh` 複製至 dirPaths.data 資料夾中

    ```bash
    cp ./docker-for-mac-openvpn/setup.sh /tmp/docker-for-mac-openvpn/setup.sh
    ```

7. 將下載的 `Development Toolkit for Kubernetes on Docker for Mac` 打包為 helm chart

    ```bash
    helm package docker-for-mac-openvpn
    ```

8. 使用剛剛打包的 helm chart 安裝 `docker-for-mac-openvpn`

    ```bash
    helm install docker-for-mac-openvpn-0.1.0.tgz -n docker-for-mac-openvpn
    ```

9. 編緝 `docker-for-mac-kubernetes-devkit/values.yaml` 中l dirPaths.local 資料夾中的 `docker-for-mac.ovpn`

    > `docker-for-mac.ovpn` 會在 VPN Server 成功建立後生成；請依需求加入想要連線的 container ip route

    ```txt
    route 172.17.0.0 255.255.0.0
    ```

10. 下載並安裝 `Tunnelblick`

    > 這是 VPN Client；請至 [Tunnelblick](https://tunnelblick.net/downloads.html) 下載，並執行安裝

11. 設定 `Tunnelblick`

    > 啟動 `Tunnelblick` 後，將上面動作編緝的 `docker-for-mac.ovpn` 牽曳至 `Tunnelblick` 的 VPN 設定列表中

    ![3addvpn](https://user-images.githubusercontent.com/3851540/71323785-32320780-2512-11ea-844c-67786c33523b.png)

    ![4vpnlist](https://user-images.githubusercontent.com/3851540/71323786-32320780-2512-11ea-8686-d5989d853a31.png)

    ![5dockerfoemac](https://user-images.githubusercontent.com/3851540/71323787-32ca9e00-2512-11ea-89fe-6a11b102f99a.png)

12. 啟動 VPN

    ![6connect](https://user-images.githubusercontent.com/3851540/71323788-32ca9e00-2512-11ea-8d86-9f0c1e636df3.png)

    ![7setuping](https://user-images.githubusercontent.com/3851540/71323789-32ca9e00-2512-11ea-8a13-17a0a8481513.png)

    ![8connected](https://user-images.githubusercontent.com/3851540/71323790-32ca9e00-2512-11ea-9580-699db09cbbe6.png)

13. 測試連線

    ![9pingok](https://user-images.githubusercontent.com/3851540/71323791-33633480-2512-11ea-879c-47bda43cc8bc.png)

    ![10rdmok](https://user-images.githubusercontent.com/3851540/71323792-33633480-2512-11ea-9a92-4897c97d7b05.png)

## 心得

今天回過頭做紀錄才發現步驟並不多，但前面我卻花了好多時間才試出一個確認可行的方法XD 雖然我自己也覺得這個解決方式有不少缺點：一定要用 Kubernetes、Helm，無形中技術門檻高了不少，再來是步驟有些繁瑣，不過至少是個可以解決問題的方法，雖不滿意但還可接受，慢慢找時間再研究改善囉，baby step 的學習也是種進步嘛

## 參考資訊

1. [Networking features in Docker Desktop for Mac - Known limitations, use cases, and workarounds](https://docs.docker.com/docker-for-mac/networking/#known-limitations-use-cases-and-workarounds)
2. [使用 docker 建立 Redis Cluster - 更新版](/redis-cluster-docker/)
3. [在Docker for Mac 上啟用Kubernetes](/docker-for-mac-kubernetes/)
4. [在 Docker for Mac 啟用 Kubernetes 後安裝 Helm](/docker-mac-kubernetes-helm/)
5. [Docker for Mac 容器网络的直连方法](https://pjw.io/articles/2018/04/25/access-to-the-container-network-of-docker-for-mac/)
6. [MAC DOCKER NETWORK TUNNEL](http://www.scispike.com/blog/mac-docker-network-tunnel/)
7. [Development Toolkit for Kubernetes on Docker for Mac](https://github.com/pengsrc/docker-for-mac-kubernetes-devkit)
