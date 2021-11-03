---
title: "修改 Docker 中 container 的 Port 對應"
date: 2017-07-18T23:30:00+08:00
lastmod: 2021-11-03T23:30:49+08:00
draft: false
tags: ["Docker"]
slug: "change-container-port-mapping"
aliases:
    - /2017/07/change-container-port-mapping.html
---
## 修改 Docker 中 container 的 Port 對應

之前和同事一起架設的 Docker container，因為環境問題，想要修改 host 對應至 container 內部的 port，原本 host 6379 port 對應至 container 的 6379 port 打算調整為 host 6380 port 對應至內部 container 的 6379 port

仔細想想，其實都用 Docker 了，何必修改現在 container port 對應，直接起一個新的 container 換掉原本的就好呀，既快速也符合 Docker 的使用邏輯。但不論做法，原本覺得修改 port 對應是個很容易的需求，想不到查了資料並不如預期中簡單，筆記一下避免忘記。

下列就將兩者做法列出，依不同情境可以選擇不同做法

## 現況

![1now](https://user-images.githubusercontent.com/3851540/28325399-477c82c2-6c10-11e7-99cd-3e22482b2a3b.png)

## 重建 Container

> 以下方式主要適用於 container 已經過修改，如果未經修改，可直接使用原始 image 重建 container 即可(步驟 3)

1. 將目標 container 停止

    > `Docker stop redis`

2. 使用目標 conatiner 現況建立 image

    > `Docker commit redis newredis`

3. 使用新建 image 重新建立 container 並指定所需的 port

    > `Docker run -d -p 6380:6379 newredis`

## 修改既有 Container

1. 將目標 container 停止

    > `Docker stop redis`

2. 修改 container 的 `hostconfig.json` 設定檔

    * Linux 檔案位置

        > `/var/lib/Docker/containers/{container_id}/hostconfig.json`

    * Windows 檔案位置

        > `C:\ProgramData\Docker\containers\{container_id}\hostconfig.json`

    * 修改對應的 port

        ```json
        .......
        "PortBindings": {
            "6379/tcp": [
                    {
                    "HostIp": "",
                    "HostPort": "6379"//改為 "6380"
                    }
                ]
            },
        .......
        ```

3. 重啟 Docker engine (清除 Docker engine cahche 設定)

    > `service Docker restart`

4. 重新啟動 container

    > `Docker start redis`

5. 效果

    ![2result](https://user-images.githubusercontent.com/3851540/28325400-477d6a0c-6c10-11e7-9c16-11d874ce2cb5.png)

## 心得

直接重建新的 container，比較符合 container 的設計及使用模式，如果 container 已有需要保存的修改，本來就該 commit 建立新的 image，再使用新 image 來建立新 container 同時重新設定 port 對應才是正統作法。

只是操作思維上還是停留在非 container 時代，一時間想法沒辦法轉過來，怕影響資料或是設定之類的，才會打算修改現有 container 的對應。但定神細想，資料或是設定本來就不該放在 container 中，或者修改後本來就該 commit 避免資料遺失

雖然是簡單的操作，卻隱含著背景思維模式的衝擊呀

## 參考資訊

1. [Reloading or Restarting the Docker Engine](https://docs.oracle.com/cd/E37670_01/E75728/html/section_yqk_dqg_gp.html)
2. [How do I assign a port mapping to an existing Docker container?](https://stackoverflow.com/questions/19335444/how-do-i-assign-a-port-mapping-to-an-existing-Docker-container)
3. [Docker Change Port Mapping for an Existing Container](https://mybrainimage.wordpress.com/2017/02/05/Docker-change-port-mapping-for-an-existing-container/)
