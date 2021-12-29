---
title: "不啟動 container 複製檔案到 host 上"
date: 2021-12-29T00:30:00+08:00
lastmod: 2021-12-29T00:30:31+08:00
draft: false
tags: ["docker"]
slug: "docker-copy-file"
---

## 不啟動 container 複製檔案到 host 上

從 container 複製檔案到 host 上，這樣的需求不算少見但也不到常用的地步，畢竟如果有常態性取得 container 中檔案的需求就會透過 volume mount 的方式來處理

不過今天想要紀錄的有些不同：一般情況下  是從 running container 將檔案複製至 host，但手上的專案不需要實際啟動 containter 只需要將 image 中的原始檔案取出即可，當然還是可以啟動 container，只是有些 container 的 init 過程很長又有啟動失敗的風險，所以今天要紀錄的是在不啟動 container 而將檔案複製到 host 上

## 基本環境脫明

1. macOS Monterey 12.0.1
2. docker desktop 3.6.0(67351)
3. docker images

    - mcr.microsoft.com/dotnet/samples:aspnetapp

## 使用方式

- 語法：使用 container id

    ```bash
    id=$(docker create {image-name}) &&
    docker cp $id:{file path and name on container} {file path and name on host} &&
    docker rm -v $id
    ```

- 範例

    ```bash
    id=$(docker create mcr.microsoft.com/dotnet/samples:aspnetapp) &&
    docker cp $id:/app/. test/ &&
    docker rm -v $id
    ```

- 另種寫法：使用 container name

    ```bash
    docker create --name yowkotest mcr.microsoft.com/dotnet/samples:aspnetapp &&
    docker cp yowkotest:/app/. test/ &&
    docker rm yowkotest
    ```

## 心得

仔細看看其實跟從 running container 複製檔案也沒差多少，只差在 container 沒有 running，可以減少 init 的時間以及出錯機會

`docker run` = `docker create` + `docker start`

如果只是要複製原本 image 中的檔案就可以使用今天紀錄的用法，但如果需要執行階段的資料就不適合了

## 參考資訊

1. [Docker - how can I copy a file from an image to a host?](https://stackoverflow.com/a/31316636)
2. [Is it possible to copy a file out of a docker image without actually running the container from that image](https://stackoverflow.com/a/48265968)
