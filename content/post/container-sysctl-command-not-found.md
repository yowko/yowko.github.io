---
title: "Container 中沒有 sysctl 指令怎麼看 kernel 設定"
date: 2020-09-24T21:30:00+08:00
lastmod: 2020-09-24T21:30:31+08:00
draft: false
tags: ["Docker","Container","Kubernetes","Linux"]
slug: "container-sysctl-command-not-found"
---

## Container 中沒有 sysctl 指令怎麼看 kernel 設定

這幾天因為某個 issue 需要檢查 linux kernel 的設定，但使用的 base image (mcr.microsoft.com/dotnet/core/aspnet:3.1) 沒有 `sysctl` 指令可以用，又找不到安裝的方式，幸虧在同事的強力支援下指點了一條明路

雖然用法簡單，但 SRE 需要的指令實在太多，加上這個語法又不是很常用到，過幾周一定會忘，快速筆記一下囉

## 基本環境設定

1. CentOS 7
2. docker 19.03.7
3. Kubernetes v1.17.8
4. docker images

    - mcr.microsoft.com/dotnet/core/aspnet:3.1
    - yowko/healthcheck:healthy

## 錯誤訊息

1. 訊息內容

    ```bash
    bash: sysctl: command not found
    ```

2. 錯誤截圖

    ![error](https://user-images.githubusercontent.com/3851540/94116188-4a59fd00-fe7d-11ea-989e-4fd6aab66000.png)

## 解決方法

1. 先找到 containerr 運行的 kubernetes node

    ```bash
    kubectl get pod -o wide
    ```

    ![2kubectlpod](https://user-images.githubusercontent.com/3851540/94116192-4b8b2a00-fe7d-11ea-8e2a-ff82fb0d420d.png)

2. 切換至 container 運作的 node 上並使用 docker 指令找出 container_name 或是 container_id

    ```bash
    docker ps
    ```

    ![3dockerps](https://user-images.githubusercontent.com/3851540/94116193-4c23c080-fe7d-11ea-8d4d-3c92d3796735.png)

3. docker 指令找出 container 的 PID (process id)

    - 語法

        ```bash
        docker inspect -f '{{.State.Pid}}' {container_name or container_id}
        ```

    - 範例

        ```bash
        docker inspect -f '{{.State.Pid}}' d0a9aa881eb9
        ```

        ```bash
        docker inspect -f '{{.State.Pid}}' k8s_checkhealthy_checkhealthy_default_76c99c1b-0122-49e0-8887-2228716045c8_0
        ```

        ![4findpid](https://user-images.githubusercontent.com/3851540/94116197-4cbc5700-fe7d-11ea-8cde-346eb824379f.png)

4. 使用 `nsenter` 對 container 下指令

    - 語法

        ```bash
        nsenter -t {PID} -n {所需指令}
        ```

    - 範例

        ```bash
        nsenter -t 22770 -n sysctl -a
        ```

        ![5nsenter](https://user-images.githubusercontent.com/3851540/94116198-4d54ed80-fe7d-11ea-971f-09ee879ef1e6.png)

## 心得

使用當下到現在筆記，我還是相當佩服同事怎麼知道這個語法的 (想必也是經過一段心酸血淚吧  哈)

不過我也聯想到：假設 kubernetes 的 container engine 不再使用 docker，之後的 container engine 也可以用相同方式嗎？

## 參考資訊

1. [Docker: any way to list open sockets inside a running docker container?](https://stackoverflow.com/questions/40350456/docker-any-way-to-list-open-sockets-inside-a-running-docker-container)
