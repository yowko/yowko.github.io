---
title: "使用 goproxy"
date: 2021-08-19T12:30:00+08:00
lastmod: 2021-08-19T12:30:31+08:00
draft: false
tags: ["Netowrk"]
slug: "goproxy"
---

## 使用 goproxy

之前筆記 [安裝 Redsocks](/redsocks) 介紹到如何使用 Redsocks 來處理特定 request 需要導向上游 proxy 的情境，也提到因為 `Squid` 不支援 https 轉 http 、`Mitmproxy` 不支援 transparent 與 upstream 並行，所以當時選擇使用 Redsocks

既然 Redsocks 這麼好，怎麼會出現今天的筆記呢？這是因為後來我們的服務使用了 GKE 雲端 Kubernetes，而 GKE 使用的 nodes 預設都採用 Container-Optimized OS，在 Container-Optimized OS 中沒有找到好的方式來安裝工具跟調整 iptable，當時也沒找到 Redsocks 的 image 所以同事才又找了 goproxy 這個工具，並且使用 image 做部署

關於 goproxy 詳細內容可以參考 GitHub-[snail007/goproxy](https://github.com/snail007/goproxy)
與 官方文件-[GOPROXY](https://snail007.github.io/goproxy/manual/zh/)

## 基本環境說明

1. macOS Big Sur 11.5.1
2. docker desktop 3.3.0(62916)
3. docker images

    - snail007/goproxy:v11.0

## 使用方式

1. 安裝方式
    - 提供 shell 快速安裝：[Quick installation](https://github.com/snail007/goproxy#quick-installation)
    - 下載 release 檔手動安裝：[Manual installation](https://github.com/snail007/goproxy#manual-installation)
    - 使用 docker

        > 個人偏好這個方式

2. 依需要的 protocol 啟動

    > goproxy 啟動時使用 `-p` 監聽 `33080`

    - http

        > `http` 指令將 gorpoxy 啟用 http proxy 模式

        ```bash
        docker run --rm -d -p 33080:33080 snail007/goproxy http -p :33080
        ```

    - sps

        >  `sps` 指令可以啟用 protocol 轉換功能 (這邊我用在 https 轉 http)，支援的 protocol 可以參考官方網 [GOPROXY](https://snail007.github.io/goproxy/manual/zh/) 說明

        ```bash
        docker run --rm -d -p 33080:33080 snail007/goproxy sps -p :33080
        ```

    - socks5

        > `socks` 指令將 gorpoxy 啟用 SOCKS5 proxy 模式，`-t` 指定 local protocol 為 `tcp` (支援 `tls`、`tcp`、`kcp`、`ws`、`wss`)

        ```bash
        docker run --rm -d -p 33080:33080 snail007/goproxy socks -t tcp -p :33080
        ```

3. 可以多重串連

    > `http` 指令將 gorpoxy 啟用 http proxy 模式，`-T` 指定上游 proxy protocol 為 `tcp`，`-p` 指定上游 proxy endpoint

    ```bash
    docker run --rm -it -p 33080:33080 snail007/goproxy http -t tcp -p :33080 -T tcp -P "22.22.22.22:8080"
    ```

## 心得

goproxy 功能面的強大是無庸置疑的，我想抱怨的是命名： goproxy 很容易讓人與加速 go module 下載的工具搞混，go 在關鍵字搜尋上就很容易出現不同面向的內容，goproxy 再補上一刀，讓你更難找到想要的資訊XD

## 參考資訊

1. [GitHub:snail007/goproxy](https://github.com/snail007/goproxy)
2. [GOPROXY](https://snail007.github.io/goproxy/manual/zh/)
3. [DockerHub:snail007/goproxy](https://hub.docker.com/r/snail007/goproxy)
