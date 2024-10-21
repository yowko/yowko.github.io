---
title: "將 Docker Multi-arch image 搬移至另個 registry 以 Nexus Repository 為例"
date: 2024-10-18T00:30:00+08:00
lastmod: 2024-10-18T00:30:31+08:00
draft: false
tags: ["bash","docker"]
slug: "nexus-docker-multi-arch-pull-push"
---

## 將 Docker Multi-arch image 搬移至另個 registry 以 Nexus Repository 為例

這個需求也是因為團隊 Nexus Repository server 的用量太高，造成服務中斷，進而影響到 CI/CD 流程，團隊的開發進度也多少受到影響，所以決定啟用多個 Nexus Repository server 以分散 server 的 loading，所以我們需要將原本的套件上傳到新的 Nexus Repository server，這篇筆記將會紀錄如何在 Nexus Repository 的 Docker Registry 下載與上傳 Docker Multi-arch image。

因為之前筆記 [將 Docker image 搬移至另個 registry 以 Nexus Repository 為例](/nexus-docker-pull-push) 沒有處理到 Multi-arch 的狀況，導致團隊在 macOS 上開發時遇到了問題

## 基本環境說明

- macOS Sequoia 15.0.1 (Apple M2 Pro)
- OrbStack 1.7.5 (18165)
- container images

     - sonatype/nexus3:3.73.0

- 使用 docker-compose 來建立 Nexus Repository

    - 8081 是 ui port
    - 8082 是 connector for docker port

    {{<gist yowko 840d60de28d5df6bbbd3e195cf778bbc "docker-compose.yml">}}

- docker registry 設定請參考之前筆記 [使用 Nexus Repository 建立 Docker Registry](/nexus-docker-registry)

## 搬移 Docker image：使用 docker buildx imagetools

1. 從 Nexus 取得所有 image 與 tag 並 push 到新的 registry
    - bash 內容

    {{<gist yowko ce5416743a06e751eaf936a17d4b3b66 "migrate-multi-arch.sh">}}

    - 使用方式

        - 語法

             ```bash
            bash migrate-docker.sh -s=http://{source username}:{source password}@{source domain}:{source port}/ -d=http://{target username}:{target password}@{target domain}:{target port}/
            ```

        - 範例

            ```bash
            bash migrate-docker.sh -s=http://admin:pass.123@localhost:8082/ -d=http://admin:pass.123@localhost:8085/
            ```

2. 錯誤重試 (非必要)

     - bash 內容

    {{<gist yowko ce5416743a06e751eaf936a17d4b3b66 "retry.sh">}}

    - 使用方式

        ```bash
        bash retry.sh
        ```

## 心得

與 [將 Docker image 搬移至另個 registry 以 Nexus Repository 為例](/nexus-docker-pull-push) 相較，使用 docker buildx imagetools 明顯在執行速度上差很多

我觀察了一下 docker buildx imagetools 的執行過程，原則上也是 pull image、tag image、push image，但因為要是  multi-arch image，所以 pull tag push 都會執行多次，加上為了要支援 multi-arch image，還要額外處理 manifest，所以執行速度上會比較慢

另外，不清楚是 Nexus 在 docker buildx 的轟炸下比較容易出現異常，還是 docker buildx 穩定度不足，實際使用上 push 出現錯誤的機率比較高，所以我有寫了一個 retry.sh 來處理這個問題

完整程式碼請參考 [GitHub:yowko/nexus-migrate](https://github.com/yowko/nexus-migrate)

## 參考資料

1. [使用 Nexus Repository 建立 Docker Registry](/nexus-docker-registry)
2. [將 Docker image 搬移至另個 registry 以 Nexus Repository 為例](/nexus-docker-pull-push)
3. [GitHub:yowko/nexus-migrate](https://github.com/yowko/nexus-migrate)
