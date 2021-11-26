---
title: "透過 docker compose 啟動 RabbitMQ cluster"
date: 2021-11-26T00:30:00+08:00
lastmod: 2021-11-26T00:30:31+08:00
draft: false
tags: ["RabbitMQ","docker"]
slug: "docker-compose-rabbitmq-cluster"
---

## 透過 docker compose 啟動 RabbitMQ cluster

最近在驗證整合 RabbitMQ cluster 的相關功能，常常要重新建立 RabbitMQ cluster，不過直接重新安裝 RabbitMQ cluster 還是覺得耗時，所以興起想要透過 docker compose 快速啟動 RabbitMQ cluster 的想法，簡單紀錄嘗試過程備忘

## 基本環境說明

1. macOS Big Sur 11.6
2. docker desktop 3.6.0(67351)
3. docker images

    - rabbitmq:3.9.10-management

## 設定方式

1. Dockerfile

    ```yaml
    FROM rabbitmq:management
    COPY cluster-entrypoint.sh /usr/local/bin/
    ENTRYPOINT /usr/local/bin/cluster-entrypoint.sh
    ```

2. cluster-entrypoint.sh

    ```bash
    #!/bin/bash

    # 在背景啟動 rabbitmq service
    /usr/local/bin/docker-entrypoint.sh rabbitmq-server -detached
    # 等待 rabbitmq 啟動完畢
    sleep 5s
    # 停止 rabbitmq application
    rabbitmqctl stop_app
    # 加入 rabbimtq cluster
    rabbitmqctl join_cluster rabbit@rabbitmq-1
    # 啟動 rabbitmq application 來 sync 資料
    rabbitmqctl start_app
    # 停止 rabbitmq service
    rabbitmqctl stop
    # 等待 rabbitmq service 完全停止
    sleep 2s
    # 將 rabbitmq service 啟動在前景
    rabbitmq-server
    ```

3. docker-compose.yml

    ```yaml
    version: '3'

    services:
    
      rabbitmq-1:
        image: rabbitmq:3.9.10-management
        hostname: rabbitmq-1
        environment:
          - RABBITMQ_DEFAULT_VHOST=/
          - RABBITMQ_DEFAULT_USER=admin
          - RABBITMQ_DEFAULT_PASS=pass.123
          - RABBITMQ_ERLANG_COOKIE=yowkotest
        ports:
          - "5672:5672"
          - "15672:15672"
    
      rabbitmq-2:
        # 如果需要調整 rabbitmq 的設定，可以在調整完 cluster-entrypoint.sh 後，將下方三行取消註解，自行 build image
        # build: 
        #   context: ./
        #   dockerfile: Dockerfile
        image: yowko/rabbitmq:3.9.10-management
        hostname: rabbitmq-2
        environment:
          - RABBITMQ_ERLANG_COOKIE=yowkotest
        depends_on:
          - rabbitmq-1
        ports:
          - "5673:5672"
          - "15673:15672"
    
      rabbitmq-3:
        image: yowko/rabbitmq:3.9.10-management
        hostname: rabbitmq-3
        environment:
          - RABBITMQ_ERLANG_COOKIE=yowkotest
        depends_on:
          - rabbitmq-1
        ports:
          - "5674:5672"
          - "15674:15672"
    ```

4. 使用方式

    - 自行 build

        ```bash
        docker-compose up --build -d
        ```

    - 直接使用

        ```bash
        docker-compose up -d
        ```

## 心得

我在執行當下遇過 `rabbitmq-2` 在啟動並成功加入 cluster 後又 shutdown 的狀況，推測是啟動時等待的時間不夠長造成的，不過發生機會不高就不直接延長等待時間，如果真的遇到無法啟動，重新啟動即可

完整原始碼: [yowko/rabbitmq-cluster](https://github.com/yowko/rabbitmq-cluster)

## 參考資訊

1. [RabbitMQ official image](https://hub.docker.com/_/rabbitmq)
2. [yowko/rabbitmq-cluster](https://github.com/yowko/rabbitmq-cluster)
