---
title: "Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image"
date: 2019-05-02T21:30:00+08:00
lastmod: 2019-05-02T21:30:31+08:00
draft: false
tags: ["Fluentd","Docker"]
slug: "fluentd-elasticsearch-docker"
---
# Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image

近期專案的 log 集中化採用 EFK - Elasticsearch + Fluentd + Kibana (log parser 改用 Fluentd 而非 Logstash 主要是因為 Logstash 有 memory 使用量大的問題)，這幾天發現設定上有些問題導致資料不如預期，於是我就開始 debug 了。

之前環境建置是由同事負責，因此在開始 debug 前就必需先架設環境接著才能重現錯誤，其中一個核心步驟：把 Fluentd 蒐集到得資料餵給 Elasticsearch 需要透過 Fluentd 的 Elasticsearch Output Plugin 才行，加上現在團隊的環境都由 docker 組成， Fluentd 官方又沒有提供對應的 image，所以我就來紀錄一下可以怎麼製作已經安裝 Elasticsearch Output Plugin 的 Fluentd base image 吧

## 基本環境說明
1. macOS Mojave 10.14.4
2. Docker Engine - Community 18.09.2
3. fluent/fluentd:edge (1.4.2)


## 方法一：fluentd GitHub 提供的 Dockerfile

> 詳細內容請參考 fluentd 官方 GitHub [Fluentd Docker Image](https://github.com/fluent/fluentd-docker-image#3-customize-dockerfile-to-install-plugins-optional)


1. Alpine version

    ```
    FROM fluent/fluentd:v1.4-2

    # Use root account to use apk
    USER root

    # below RUN includes plugin as examples elasticsearch is not required
    # you may customize including plugins as you wish
    RUN apk add --no-cache --update --virtual .build-deps \
            sudo build-base ruby-dev \
    && sudo gem install fluent-plugin-elasticsearch \
    && sudo gem sources --clear-all \
    && apk del .build-deps \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

    COPY fluent.conf /fluentd/etc/
    COPY entrypoint.sh /bin/

    USER fluent
    ```


2. Debian version

    ```
    FROM fluent/fluentd:v1.4-debian-2

    # Use root account to use apt
    USER root

    # below RUN includes plugin as examples elasticsearch is not required
    # you may customize including plugins as you wish
    RUN buildDeps="sudo make gcc g++ libc-dev" \
    && apt-get update \
    && apt-get install -y --no-install-recommends $buildDeps \
    && sudo gem install fluent-plugin-elasticsearch \
    && sudo gem sources --clear-all \
    && SUDO_FORCE_REMOVE=yes \
        apt-get purge -y --auto-remove \
                    -o APT::AutoRemove::RecommendsImportant=false \
                    $buildDeps \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

    COPY fluent.conf /fluentd/etc/
    COPY entrypoint.sh /bin/

    USER fluent
    ```

## 方法二：使用 fluent-gem (推薦)

> 這是從 Fluentds 官網 [Elasticsearch Output Plugin](https://docs.fluentd.org/v1.0/articles/out_elasticsearch) 看來的做法，我覺得語法很簡潔就試著在 dockerfile 中使用

1. 基本用法

    ```
    FROM fluent/fluentd:edge
    USER root
    RUN fluent-gem install fluent-plugin-elasticsearch
    USER fluent
    ```

    > USER 切換是參考 Fluentd GitHub 的做法，否則會出現 no permission 的錯誤

2. 錯誤範例

    - Dockerfile 未切換 User 

        ```
        FROM fluent/fluentd:edge
        RUN fluent-gem install fluent-plugin-elasticsearch
        ```

    - 錯誤訊息

        - 訊息內容

            ```
            ERROR:  While executing gem ... (Gem::FilePermissionError)
                You don't have write permissions for the /usr/lib/ruby/gems/2.5.0 directory.
            ```

        - 錯誤截圖

            ![1error](https://user-images.githubusercontent.com/3851540/57087241-3797d300-6d32-11e9-82bc-c139aeab8539.png)

3. 個人使用版本

    > 加上 expose port 可以用來處理 forward 或是接受 http request

    ```
    FROM fluent/fluentd:edge
    USER root
    RUN fluent-gem install fluent-plugin-elasticsearch
    USER fluent
    EXPOSE 24224 24224/udp 8888 
    ```

## 心得

我在查資料時覺得很困惑，看不出哪個做法才是對的，GitHub 是官方帳號，而與官網做法又不同，最後決定都試試後再說，以結果來看這次選擇是正確的：語法單純很多，也順利完成目的，分享給大家囉

原始 Dockerfile：[yowko/Fluentd-elasticsearch-dockerfile](https://github.com/yowko/Fluentd-elasticsearch-dockerfile)

# 參考資訊
1. [Fluentd Docker Image](https://github.com/fluent/fluentd-docker-image#3-customize-dockerfile-to-install-plugins-optional)
2. [Elasticsearch Output Plugin](https://docs.fluentd.org/v1.0/articles/out_elasticsearch)