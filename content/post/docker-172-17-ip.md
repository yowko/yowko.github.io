---
title: "Docker 無法連線至特定網段 (172.17.x.x)"
date: 2017-09-02T17:00:00+08:00
lastmod: 2021-11-03T17:00:14+08:00
draft: false
tags: ["Docker","Linux"]
slug: "docker-172-17-ip"
aliases:
    - /2017/09/docker-172-17-ip.html
---
## Docker 無法連線至特定網段 (172.17.x.x)

公司正逐步採用 docker 來進行快速部署，一開始先從 Jenkins、Redis 這類基礎服務著手，在開發環境一切都很順利，加上 docker 的部署速度，讓整個計劃倍受好評，沒想到在 stage 環境卻出現意料外的狀況：docker 建立的服務與 docker host 都無法連線至 `172.17` 開頭的 server

## 重現問題

1. 無法連線至 `172.17`

    ![117217](https://user-images.githubusercontent.com/3851540/29993769-0afbd1a2-8ff3-11e7-83ea-e30f34c2bcc8.png)

2. 連線 `172.16` 則一切正常

    ![211716](https://user-images.githubusercontent.com/3851540/29993770-0b0a6a96-8ff3-11e7-9594-4b8ee7dfb229.png)

## 如何確認是 docker 造成的

1. 在 docker host 上找出 docker 的 bridge

    ```bash
    brctl show
    ```

    ![3brctl](https://user-images.githubusercontent.com/3851540/29993764-0ad736da-8ff3-11e7-9e09-2ae53dfcea8b.png)

2. 將 docker 的 bridge 停用

    ```bash
    ifconfig docker0 down
    ```

3. 重新測試是否可以連線至目標 server

    ![4downping](https://user-images.githubusercontent.com/3851540/29993763-0ad31eb0-8ff3-11e7-94fc-0d83d1d0355c.png)

    > 如果已可正常連線，基本上已經可以確認是 docker 所造成的問題，以下的 `step 4` 與 `step 5` 是用來協助再次確認問題是 docker 造成的，非必要執行

4. 重複驗證 - 將 docker bridge0 啟用

    ```bash
    ifconfig docker0 up
    ```

5. 確認是否又無法正常連線

    ![5upping](https://user-images.githubusercontent.com/3851540/29993767-0ade4cb8-8ff3-11e7-87e9-ebf02619f180.png)

## 問題發生原因

docker 在 linux 環境中預設指定給 container 使用的 ip 就是 `172.17.x.x` 開頭，因此在 container 或是 docker host 會將所有對於 `172.17.x.x` 的 request 導向 docker bridge 處理

相關細節請參考 [Customize the docker0 bridge](https://docs.docker.com/engine/userguide/networking/default_network/custom-docker0/)

## 解決方式

1. 在 `/etc/docker/` 加上 `daemon.json`

    ```bash
    vi /etc/docker/daemon.json
    ```

2. 在 `daemon.json` 指定 `bip` 範圍

    ```json
    {
        "bip": "192.168.168.1/24"
    }
    ```

    > `bip` 範圍請依實際情況調整，詳細設定請參考官方文件 [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)

3. 重啟 docker service 以套用設定

    > 只重啟 container 沒有作用。<span style="color:red">注意：重啟 dcoker service 會 shutdown 所有 container</span>

    ```bash
    service docker restart
    ```

    ![6dockerrestart](https://user-images.githubusercontent.com/3851540/29993765-0ada6bc0-8ff3-11e7-8852-d05b64154e49.png)

4. 效果

    > 使用 `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id` 來確認 container 使用的 ip，詳細介紹可以參考 [如何取得 Container 所使用的 Ip](/2017/09/container-ip.html)

    * 修改前：預設使用 `172.17.0.3`

        ![7before](https://user-images.githubusercontent.com/3851540/29993766-0adcc96a-8ff3-11e7-9da0-34068f44d58f.png)

    * 修改後：依設定使用 `192.168.168.2`

        ![8after](https://user-images.githubusercontent.com/3851540/29993768-0af84f8c-8ff3-11e7-82b3-cce7eb5b8ec5.png)

    * 已可正常存取 `172.17.x.x` 網段 server

        ![4downping](https://user-images.githubusercontent.com/3851540/29993763-0ad31eb0-8ff3-11e7-94fc-0d83d1d0355c.png)

## 心得

其實這個 issue 只有在 docker 需要與 `172.17.x.x` 網段 server 互動時才會出現，但剛好公司不少 server 都用到 `172.17.x.x` 網段，算是天時地利搭配得好才有機會踩到這個問題 @@"

雖然花了不少時間才解決，不過也多虧這次機會多了解 docker 配置 container ip 的機制，也許以後還會派上用場，多學總是好的

## 參考資訊

1. [Customize the docker0 bridge](https://docs.docker.com/engine/userguide/networking/default_network/custom-docker0/)
2. [如何取得 Container 所使用的 Ip](/2017/09/container-ip.html)
3. [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)
