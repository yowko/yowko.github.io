---
title: "Container 中執行 apt-get update 時遇到 GPG KEYEXPIRED"
date: 2020-07-26T21:30:00+08:00
lastmod: 2020-07-26T21:30:31+08:00
draft: false
tags: ["Linux","Docker"]
slug: "container-gpg-keyexpired"
---

## Container 中執行 apt-get update 時遇到 GPG KEYEXPIRED

之前筆記 [執行 apt-get update 時遇到 GPG KEYEXPIRED](https://blog.yowko.com/debian-keyexpired/) 紀錄到在使用 image : `confluentinc/cp-kafka:5.5.0` 建立 container 後需要安裝其他套件，但 apt-get update 卻遇到 GPG KEYEXPIRED 的錯誤，當時透過進到 container 中手動調整 source 的內容，但這樣一來對於常要重建環境的我來說，實在有些困擾：每次重建環境有問題，需要 debug 時就要進到 container 中手動調整，所以我興起了直接修改 image 的想法，簡單紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)
3. docker images

    - confluentinc/cp-kafka:5.5.0

        > Debian GNU/Linux 8 (jessie)

## 錯誤訊息

1. 訊息內容

    ```txt
    W: GPG error: http://archive.debian.org jessie Release: The following signatures were invalid: KEYEXPIRED 1587841717
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/87849408-faaa0300-c91a-11ea-986d-156f88459477.jpg)

## 解決方式

> 直接修改 dockerfile，在 build image 時就直接修改 source

- 原始 dockerfile

    ```dockerfile
    #
    # Copyright 2016 Confluent Inc.
    #
    # Licensed under the Apache License, Version 2.0 (the "License");
    # you may not use this file except in compliance with the License.
    # You may obtain a copy of the License at
    #
    # http://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.
    
    FROM confluentinc/cp-base
    
    ARG COMMIT_ID=unknown
    LABEL io.confluent.docker.git.id=$COMMIT_ID
    ARG BUILD_NUMBER=-1
    LABEL io.confluent.docker.build.number=$BUILD_NUMBER
    
    MAINTAINER partner-support@confluent.io
    LABEL io.confluent.docker=true
    
    # allow arg override of required env params
    ARG KAFKA_ZOOKEEPER_CONNECT
    ENV KAFKA_ZOOKEEPER_CONNECT=${KAFKA_ZOOKEEPER_CONNECT}
    ARG KAFKA_ADVERTISED_LISTENERS
    ENV KAFKA_ADVERTISED_LISTENERS=${KAFKA_ADVERTISED_LISTENERS}
    
    ENV COMPONENT=kafka
    
    # primary
    EXPOSE 9092
    
    RUN echo "===> installing ${COMPONENT}..." \
        && apt-get update && apt-get install -y confluent-kafka-${SCALA_VERSION}=${CONFLUENT_VERSION}${CONFLUENT_PLATFORM_LABEL}-${CONFLUENT_DEB_VERSION} \
        \
        && echo "===> clean up ..."  \
        && apt-get clean && rm -rf /tmp/* /var/lib/apt/lists/* \
        \
        && echo "===> Setting up ${COMPONENT} dirs..." \
        && mkdir -p /var/lib/${COMPONENT}/data /etc/${COMPONENT}/secrets\
        && chmod -R ag+w /etc/${COMPONENT} /var/lib/${COMPONENT}/data /etc/${COMPONENT}/secrets \
        && chown -R root:root /var/log/kafka /var/log/confluent /var/lib/kafka /var/lib/zookeeper
    
    
    VOLUME ["/var/lib/${COMPONENT}/data", "/etc/${COMPONENT}/secrets"]
    
    COPY include/etc/confluent/docker /etc/confluent/docker
    
    CMD ["/etc/confluent/docker/run"]
    ```

- 修改後 dockerfile

    > 多了 `sed -i 's;http://archive.debian.org/debian/;http://deb.debian.org/debian/;' /etc/apt/sources.list \`

    ```dockerfile
    #
    # Copyright 2016 Confluent Inc.
    #
    # Licensed under the Apache License, Version 2.0 (the "License");
    # you may not use this file except in compliance with the License.
    # You may obtain a copy of the License at
    #
    # http://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.
    
    FROM confluentinc/cp-base
    
    ARG COMMIT_ID=unknown
    LABEL io.confluent.docker.git.id=$COMMIT_ID
    ARG BUILD_NUMBER=-1
    LABEL io.confluent.docker.build.number=$BUILD_NUMBER
    
    MAINTAINER partner-support@confluent.io
    LABEL io.confluent.docker=true
    
    # allow arg override of required env params
    ARG KAFKA_ZOOKEEPER_CONNECT
    ENV KAFKA_ZOOKEEPER_CONNECT=${KAFKA_ZOOKEEPER_CONNECT}
    ARG KAFKA_ADVERTISED_LISTENERS
    ENV KAFKA_ADVERTISED_LISTENERS=${KAFKA_ADVERTISED_LISTENERS}
    
    ENV COMPONENT=kafka
    
    # primary
    EXPOSE 9092
    
    RUN echo "===> installing ${COMPONENT}..." \
        && sed -i 's;http://archive.debian.org/debian/;http://deb.debian.org/debian/;' /etc/apt/sources.list \
        && apt-get update && apt-get install -y confluent-kafka-${SCALA_VERSION}=${CONFLUENT_VERSION}${CONFLUENT_PLATFORM_LABEL}-${CONFLUENT_DEB_VERSION} \
        \
        && echo "===> clean up ..."  \
        && apt-get clean && rm -rf /tmp/* /var/lib/apt/lists/* \
        \
        && echo "===> Setting up ${COMPONENT} dirs..." \
        && mkdir -p /var/lib/${COMPONENT}/data /etc/${COMPONENT}/secrets\
        && chmod -R ag+w /etc/${COMPONENT} /var/lib/${COMPONENT}/data /etc/${COMPONENT}/secrets \
        && chown -R root:root /var/log/kafka /var/log/confluent /var/lib/kafka /var/lib/zookeeper
    
    
    VOLUME ["/var/lib/${COMPONENT}/data", "/etc/${COMPONENT}/secrets"]
    
    COPY include/etc/confluent/docker /etc/confluent/docker
    
    CMD ["/etc/confluent/docker/run"]
    ```

## 心得

完整程式碼請參考 [yowko/cp-kafka](https://github.com/yowko/cp-kafka)

這個問題應該只是暫時性的，我猜測應該很快就會獲得修正，在那之前就先擋著先吧

## 參考資訊

1. [[cp-base] Apt-get update is broken. The following signatures were invalid: KEYEXPIRED 1587841717](https://github.com/confluentinc/cp-docker-images/issues/849#issuecomment-626771381)