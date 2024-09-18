---
title: "使用 Nexus Repository 建立 Docker Registry"
date: 2024-09-18T00:30:00+08:00
lastmod: 2024-09-18T00:30:31+08:00
draft: false
tags: ["container","docker"]
slug: "nexus-docker-registry"
---

## 使用 Nexus Repository 建立 Docker Registry

之前筆記 [使用 Docker 建立 Nexus3 的 Image Registry](/nexus-docker-image-rergistry/) 紀錄到如何使用 Sonatype Nexus Repository 來建立 Docker Image Registry，最近遇到幾次需要使用 Docker Registry 的需求，但之前筆記 [使用 Docker 建立 Nexus3 的 Image Registry](/nexus-docker-image-rergistry/) 卻因為不同因素派不上用場，所以今天就來針對最近遇到的問題來補充如何使用 Sonatype Nexus Repository 來建立 Docker Registry。

## 基本環境設定

- macOS Sonoma 14.6.1 (Apple M2 Pro)
- OrbStack 1.7.2 (17389)
- container images

     - sonatype/nexus3:3.72.0

- 使用 docker-compose 來建立 Nexus Repository

    - 8081 是 ui port
    - 8082 是 connector for docker port

    {{<gist yowko a8e14cb4f4224183dd565bc7062c7267 "docker-compose.yml">}}

## 設定方式與使用方式

1. 首次登入並完成設定

    - 首次登入 Nexus Repository 需要從 container 中取得 password，並修改密碼

        ![1password](https://github.com/user-attachments/assets/62ec14bc-9baa-42fd-9440-678d05819b08)

        ```bash
        docker exec -it nexus cat /nexus-data/admin.password 
        ```

    - 修改密碼並完成設定

        ![2setup1](https://github.com/user-attachments/assets/972b1467-4939-4ea9-8189-ca891af75bca)

        ![3setup2](https://github.com/user-attachments/assets/68f4bab3-1987-4d10-93a3-20510d17ce95)

        ![4setup3](https://github.com/user-attachments/assets/12e3b2a6-c281-4086-817d-1cd0694175eb)

        ![5setup4](https://github.com/user-attachments/assets/c93faee9-fb66-4bd1-9600-05cee4afd533)

2. 建立 Docker Registry

    - `Server administration and configuration` --> `Repositories` --> `Create repository`

        ![6createrepo](https://github.com/user-attachments/assets/a0c93269-e416-41ae-bb6e-c44358554be4)

    - Select Recipe --> `docker (hosted)`

        ![7dockerhost](https://github.com/user-attachments/assets/4e5f431c-76a2-40a0-b41c-a72c32f2e879)

    - Create Repository: docker (hosted)

        - `Name` 是必填：會影響之後 image 的完整名稱
        - `HTTP` connector port 是為了方便 docker client 使用，也會影響之後 image 的完整名稱 (設定需與 docker compose 中除了 8081 之外的 port 一致)

        ![8dockerhosted](https://github.com/user-attachments/assets/bd728025-674b-4c61-8395-d325ae33b613)

3. 允許匿名 pull 時需設定 Docker Bearer Token Realm：`Security` --> `Realms` --> 加入 `Docker Bearer Token Realm`

    ![9dockereralms](https://github.com/user-attachments/assets/972ac113-e940-4310-9302-017c04a4e289)

    - `未設定錯誤：Error response from daemon: unauthorized`

        ![10unauthorized](https://github.com/user-attachments/assets/e2778146-9590-473c-aeaf-65f79054d6b7)

4. 實際使用

    > port 使用 connector port，不是 8081

    - 登入：`docker login localhost:8082`

        ![11login](https://github.com/user-attachments/assets/a0e8a4a0-5789-4dc5-8bfb-8f5afcd54401)

    - image 名稱模版： `{host}:{port}/{repository name}/{image name}:{tag}`

        > Nexus 中的 {repository name} pattern 是 `repository/{建立時的 Name}`

    - image 名稱範例： `localhost:8082/repository/docker/redis:latest`

        ![12dockerpush](https://github.com/user-attachments/assets/fcc615af-9dd7-4f5c-8fa9-ad87a89d731c)

## 心得

Sonatype Nexus Repository 從 3.66.0 開始加入了 threshold 的機制，用量超過 `20,000 requests/每天` 或是 `100,000 components` 會收到警告，依我自己的經驗 nexus 會暫停提供服務，重啟後可以緩解，但畢竟 threshold 已經頂到了，還是會持續出現問題，所以建議在使用前要先評估用量，避免出現問題。

## 參考資訊

1. [使用 Docker 建立 Nexus3 的 Image Registry](/nexus-docker-image-rergistry/)
2. [Docker Authentication](https://help.sonatype.com/en/docker-authentication.html)
