---
title: "Windows Server 2016 中 Windows Container 的 docker 指令介紹"
date: 2017-05-10T23:29:00+08:00
lastmod: 2018-09-18T23:29:22+08:00
draft: false
tags: ["Docker","Windows Server 2016"]
slug: "docker-command-8n-windows"
aliases:
    - /2017/05/docker-command-in-windows.html
---
# Windows Server 2016 中 Windows Container 的 docker 指令介紹
經過幾天筆記加深印象後，大致上對於 container 與 docker 有些基本了解，接著就來看看該如何在 Windows Server 2016 中使用 docker 及 docker 相關指令

## Container 生命周期相關語法

> 指定 container 時只要使用可以識別的 id 簡碼即可，

*   e.g. 下列只要沒有跟其他 container 重複都是有效的


    *   id：`43abd063155090265f694bc4708d606667440316048bb76a131b097d9a0f8e77`
    *   `43abd0631550`
    *   `43`
    *   `4`

        ![2uniqueid](https://cloud.githubusercontent.com/assets/3851540/25906007/1b07ec42-35d6-11e7-8465-b8c3b4d15d35.png)

1.  [docker create](https://docs.docker.com/engine/reference/commandline/create/)

    > 建立一個 container 但是不啟動

    - 語法
        
        >docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
    - 範例
        
        ```
        docker create microsoft/iis
        ```

2.  [docker rename](https://docs.docker.com/engine/reference/commandline/rename/)

    > 重新命名 container

    - 語法
     
        > docker rename CONTAINER NEW_NAME
    - 範例
        
        ```
        docker rename 43 iis
        ```

3.  [docker run](https://docs.docker.com/engine/reference/commandline/run/)

    > 建立並啟動一個 container

    - 語法
        
        >   docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    - 範例
        
        ```
        docker run -d microsoft/iis
        ```

4. [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)

    > 刪除容器 container

    - 語法
        
        >   docker rm [OPTIONS] CONTAINER [CONTAINER...]

    - 範例
     
        ```
        docker rm 43
        ```
5.  [docker update](https://docs.docker.com/engine/reference/commandline/update/)

    > 更新 container 的硬體設定

    - 語法
        
        >   docker update [OPTIONS] CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker update 43
        ```

6.  [docker start](https://docs.docker.com/engine/reference/commandline/start/)

    > 啟動 container

    - 語法
        
        >   docker start [OPTIONS] CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker start 43
        ```

7.  [docker stop](https://docs.docker.com/engine/reference/commandline/stop/)

    > 停止執行中的 container

    - 語法
        
        >  docker stop [OPTIONS] CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker stop 43
        ```

8.  [docker restart](https://docs.docker.com/engine/reference/commandline/restart/)

    > 重啟 container

    - 語法
    
        >   docker restart [OPTIONS] CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker restart 43
        ```

9.  [docker pause](https://docs.docker.com/engine/reference/commandline/pause/)

    > 暫停執行中的 container，將其 "凍結" 在當前狀態

    - 語法
         
        > docker pause CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker pause 43
        ```

10. [docker unpause](https://docs.docker.com/engine/reference/commandline/unpause/)

    > 結束 container 暫停狀態

    - 語法
        
        >  docker unpause CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker unpause 43
        ```

11.  [docker wait](https://docs.docker.com/engine/reference/commandline/wait/)

    > 等待到執行中的 container 停止為止

    - 語法
        
        >  docker wait CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker wait 43
        ```

12.  [docker kill](https://docs.docker.com/engine/reference/commandline/kill/)

    > 向執行中的 container 傳送 SIGKILL 指令, 單就效果來看跟 docker stop 一樣

    - 語法
        
        >  docker kill [OPTIONS] CONTAINER [CONTAINER...]
    - 範例
        
        ```
        docker kill 43
        ```

13.  [docker attach](https://docs.docker.com/engine/reference/commandline/attach/)

    > 掛載至執行中的 container

    - 語法
        
        >  docker attach [OPTIONS] CONTAINER
    - 範例
        
        ```
        doocker attch 43
        ```

## Container 資訊相關語法

1.  [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)

    > 查看運行中的所有 container

    - 語法
        
        > docker ps [OPTIONS]

    - 範例
        
        - `docker ps`

            > 查看所有執行中的 container
        - `docker ps -a`

            > 查看所有執行中的和已停止的 container

2.  [docker logs](https://docs.docker.com/engine/reference/commandline/logs/)

    > 從 container 中取得執行狀態 log

    - 語法
    
        >  docker logs [OPTIONS] CONTAINER
    - 範例
        
        ```
        docker logs 43
        ```

3.  [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/)

    > 查看某個 container 的所有資訊(包括網路、id、path、volume....)

    - 語法
        
        >   docker inspect [OPTIONS] NAME|ID [NAME|ID...]
    - 範例
        
        ```
        docker inspect 43
        ```

4.  [docker events](https://docs.docker.com/engine/reference/commandline/events/)

    > 監聽 container 當下發生的 events

    - 語法
        
        >    docker events [OPTIONS]
    - 範例
        
        ```
        docker events
        ```

5.  [docker port](https://docs.docker.com/engine/reference/commandline/port/)

    > 查看 container 的公開 port

    - 語法
    
        >  docker port CONTAINER [PRIVATE_PORT[/PROTO]]
    - 範例
        
        ```
        docker port 43
        ```

6.  [docker top](https://docs.docker.com/engine/reference/commandline/top/)

    > 查看 container 中執行的 process

    - 語法
    
        >  docker top CONTAINER [ps OPTIONS]
    - 範例
        
        ```
        docker top 43
        ```

7.  [docker stats](https://docs.docker.com/engine/reference/commandline/stats/)

    > 查看 container 的資源使用情況統計資訊

    - 語法
        
        >   docker stats [OPTIONS] [CONTAINER...]
    - 範例
        
        - `docker stats 43`
        
            >  顯示 id 為 43 的 containers 資訊
        -  `docker stats --all `
            
            >  顯示正在執行中的 containers 資訊

8. [docker diff](https://docs.docker.com/engine/reference/commandline/diff/) (在 Windows Container 不支援針對執行中的 container 下 diff 指令)

    > 查看 dontainer的檔案系統中有異動的檔案

    - 語法
        
        >docker diff CONTAINER
    - 範例
        
        ```
        docker diff 43
        ```

## Container 互動行為相關語法

1.  [docker cp](https://docs.docker.com/engine/reference/commandline/cp/)

    > 在 host 和 container 間交換檔案

    *   docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
    *   docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

2. [docker export](https://docs.docker.com/engine/reference/commandline/export/)

    > 將 container 的檔案系統匯出成 tar 壓縮檔(Windows Container 不支援這個指令)

    - 語法
        
        >   docker export [OPTIONS] CONTAINER
    - 範例
        
        ```
        docker export -o="a.tar" 43
        ```

3.  docker exec

    > 在 container 中執行指令

    - 語法
        
        >   docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
    - 範例
        
        ```
        docker exec -it 43 ipconfig
        ```

## Image 生命周期相關語法

1.  [docker images](https://docs.docker.com/engine/reference/commandline/images/)

    > 列出所有 image

    - 語法
        
        >  docker images [OPTIONS] [REPOSITORY[:TAG]]
    - 範例

        ```
        docker images
        ```

2.  [docker import](https://docs.docker.com/engine/reference/commandline/import/)

    > 從壓縮檔中匯入 image

    - 語法
        
        >   docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
    - 範例
        
        ```
        docker import [http://example.com/exampleimage.tgz](http://example.com/exampleimage.tgz)
        ```

3.  [docker build](https://docs.docker.com/engine/reference/commandline/build/)

    > 從 Dockerfile 建立 image

    - 語法
        
        >  docker build [OPTIONS] PATH | URL | -
    - 範例
        
        ```
        docker build -t iis c:\build
        ```

4.  [docker commit](https://docs.docker.com/engine/reference/commandline/commit/)(在 Windows Container 不支援針對執行中的 container 下 commit 指令)

    > 使用 container 建立 image，建立時 container 暫停。

    - 語法
        
        >  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
    - 範例
        
        ```
        docker commit 43 yowko/test:v1.0
        ```

5.  [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)

    > 刪除 image

    - 語法
        
        >   docker rmi [OPTIONS] IMAGE [IMAGE...]
    - 範例
        
        ```
        docker rmi microsoft/iis
        ```

6.  [docker save](https://docs.docker.com/engine/reference/commandline/save/)

    > 使用 STDOUT 將 image 壓縮，包括所有的父層，tag 和 versions (0.7 起).

    - 語法
        
        >   docker save [OPTIONS] IMAGE [IMAGE...]
    - 範例
        
        ```
        docker save -o="a.tar" microsoft/iis
        ```

7.  [docker load](https://docs.docker.com/engine/reference/commandline/load/)

    > 使用 STDIN 從壓縮檔載入 images，包括 tag (0.7 起).

    - 語法
        
        >docker load [OPTIONS]
    - 範例
        
        ```
        docker load a.tar
        ```

8.  [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)

    > 從 registry 取得 image

    - 語法
        
        > docker pull [OPTIONS] NAME[:TAG|@DIGEST]
    - 範例
        
        ```
        docker pull microsoft/aspnet
        ```

## Image 資訊相關語法

1.  [docker history](https://docs.docker.com/engine/reference/commandline/history/)

    > 顯示 image 的歷程紀錄

    - 語法
        
        >    docker history [OPTIONS] IMAGE
    - 範例
        
        ```
        docker history microsoft/aspnet
        ```

2.  [docker tag](https://docs.docker.com/engine/reference/commandline/tag/)

    > 將 image 建立 tag

    - 語法
        
        > docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
    - 範例
        
        ```
        docker tag microsoft/aspnet ms/aspnet:v1.0
        ```

# 參考資訊
1.  [docker command](https://docs.docker.com/edge/engine/reference/commandline/docker/)