---
title: "在 Docker Compose file 3 下限制 CPU 與 Memory"
date: 2019-07-24T21:30:00+08:00
lastmod: 2019-07-24T21:30:31+08:00
draft: false
tags: ["Docker"]
slug: "docker-compose-3-cpu-memory-limit"
---

## 在 Docker Compose file 3 下限制 CPU 與 Memory

這幾天某個 job 在進行最終階段測試時發現用到的某個 container 會出現 cpu high 的現象，而且狀況嚴重到會讓 docker process cpu 衝到 400% 以上，接著 docker service 掛了 XD，連帶該 server 上的所有 container 都跟著陪葬了，於是興起在 container 啟動時限制 cpu 以及 memory 用量的念頭：避免因為某個功能異常拖垮其他所有服務

雖然過去就耳聞過 docker-compose version 3 開始就不再支援 `mem_limit` 與 `cpu_shares` 設定，想要針對 cpu 與 memory resource 做限制只能改用 docker swarm，但只是測試環境實際不想大費周章架 kubernetes 加上開發環境都是使用 docker-compose 來建立相關服務，為了整合測試調整開發 flow 並不合理，原本打定最差情況就是將 docker-compose file 退版 - 改用 version 2，所幸最後有找個可以接受的方法，就來看看可以怎麼做吧

## 基本環境說明

1. macOS Mojave 10.14.5
2. docker 18.09.2
3. docker-compose version 1.23.2, build 1110ad01
4. docker-compose.yml

    > 用來模擬 cpu 衝高的 container

    ```yml
    version: "3"
    services:
    redis:
        image: redis:alpine
        container_name: testredis
    ```

## 模擬 CPU 飆高

1. 使用 docker-compose 啟動 container

    > 進到 `docker-compose.yml` 的所在資料夾位置執行下列資料

    ```bash
    docker-compose up -d
    ```

2. 進行該 conatiner 中讓 cpu 飆高

    ```bash
    docker exec -it testredis sh

    for i in $(seq `grep -c ^proc /proc/cpuinfo`); do (yes > /dev/null &); done
    ```

3. 確認設定

    > 使用 `docker inspect testredis | grep Cpu` 檢查 container 相關設定

    ![2cpusetting](https://user-images.githubusercontent.com/3851540/61802721-881b5a00-ae63-11e9-8081-e17b8746692f.png)

4. 實際效果

    > 透過 `docker stats testredis` 監控 container 的使用量可以看到 cpu 從 0.33% 直線上升至超過 300%

    ![1cpuhigh](https://user-images.githubusercontent.com/3851540/61802720-881b5a00-ae63-11e9-9dfe-1aaf9e1af656.png)

## 加入 CPU 限制

1. 修改 yml

    - `version` 改用 `3.7`
    - 加入 deploy 的 resource limit

    ```yml
    version: "3.7"
    services:
      redis:
        image: redis:alpine
        container_name: testredis
        deploy:
          resources:
            limits:
              cpus: '0.50'
    ```

2. 啟動 container 時加入 `--compatibility` 參數

    ```bash
    docker-compose --compatibility up -d
    ```

    - 啟動時未加入 `-compatibility` 的警告

        ```txt
        WARNING: Some services (redis) use the 'deploy' key, which will be ignored. Compose does not support 'deploy' configuration - use `docker stack deploy` to deploy to a swarm.
        ```

3. 確認設定

    > 使用 `docker inspect testredis | grep Cpu` 檢查 container 相關設定，可以看到 `NanoCpu = 500000000 (1000000000 * 0.5)`

    ![3nanocpus](https://user-images.githubusercontent.com/3851540/61802723-88b3f080-ae63-11e9-9eb6-8641c432d02a.png)

4. 實際效果

    > 透過 `docker stats testredis` 監控 container 的使用量可以看到 cpu 從 0.3% 上升至 50% 左右就不再上升

    ![4cpu50](https://user-images.githubusercontent.com/3851540/61802724-88b3f080-ae63-11e9-8d2e-ac46f30f9712.png)

## 心得

同場加映 memory 限制

```yml
version: "3.7"
services:
  redis:
    image: redis:alpine
    container_name: testredis
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 500M

```

`--compatibility` 是 `docker-compose 1.20.0` 加入，主要目的就是用來將 `deploy` 中的 `資料限制`、`replicas` 與 `重啟策略` 直接轉譯為 version 2 的語法

雖然只是一個小小參數，但卻透露出其實我對 docker-compose 的不熟悉，花了一點時間才找到，幸虧亡羊補牢，沒有繼續錯下去

## 參考資訊

1. [How to specify Memory & CPU limit in docker compose version 3](https://stackoverflow.com/questions/42345235/how-to-specify-memory-cpu-limit-in-docker-compose-version-3)
2. [一行文：CPU壓力測試](https://2formosa.blogspot.com/2017/04/cpu-pressure-test.html)
