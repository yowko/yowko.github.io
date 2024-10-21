---
title: "將 Docker image 搬移至另個 registry 以 Nexus Repository 為例"
date: 2024-10-16T00:30:00+08:00
lastmod: 2024-10-16T00:30:31+08:00
draft: false
tags: ["bash","docker"]
slug: "nexus-docker-pull-push"
---

## 將 Docker image 搬移至另個 registry 以 Nexus Repository 為例

這個需求也是因為團隊 Nexus Repository server 的用量太高，造成服務中斷，進而影響到 CI/CD 流程，團隊的開發進度也多少受到影響，所以決定啟用多個 Nexus Repository server 以分散 server 的 loading，所以我們需要將原本的套件上傳到新的 Nexus Repository server，這篇筆記將會紀錄如何在 Nexus Repository 的 Docker Registry 下載與上傳 Docker image。

不過 docker 流程比較簡單，只需要 pull image、tag image、push image 就可以，相對單純

如果有 multi-arch image 的需求，可以參考 [將 Docker Multi-arch image 搬移至另個 registry 以 Nexus Repository 為例](/nexus-docker-multi-arch-pull-push)

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

## 搬移 Docker image

1. bash 內容

    {{<gist yowko 840d60de28d5df6bbbd3e195cf778bbc "migrate-docker.sh">}}

2. 使用方式

    - 語法

         ```bash
        bash migrate-docker.sh -s=http://{source username}:{source password}@{source domain}:{source port}/ -d=http://{target username}:{target password}@{target domain}:{target port}/
        ```

    - 範例

        ```bash
        bash migrate-docker.sh -s=http://admin:pass.123@localhost:8082/ -d=http://admin:pass.123@localhost:8085/
        ```

## 心得

主觀感受上，docker image 搬移比較簡單，只需要 pull image、tag image、push image 就可以，相對單純，不過仔細想想，整體流程與 nuget 跟 npm 也大致都相同，至於為什麼其中有不小的差異，我自己推測與一致的 api 有關，雖然 nuget cli 與 npm cli 可能也提供了 api 來下載與上傳套件，但 nuget 與 cli 因為角色是軟體套件管理器的關係，行為終究與 docker image 直接 pull 就能使用的行為有所不同，造成 docker image 的遷移相較於 nuget 與 npm 簡單的原因

如果有 multi-arch image 的需求，可以參考 [將 Docker Multi-arch image 搬移至另個 registry 以 Nexus Repository 為例](/nexus-docker-multi-arch-pull-push)

完整程式碼請參考 [GitHub:yowko/nexus-migrate](https://github.com/yowko/nexus-migrate)

## 參考資料

1. [使用 Nexus Repository 建立 Docker Registry](/nexus-docker-registry)
2. [將 Docker Multi-arch image 搬移至另個 registry 以 Nexus Repository 為例](/nexus-docker-multi-arch-pull-push)
3. [GitHub:yowko/nexus-migrate](https://github.com/yowko/nexus-migrate)
