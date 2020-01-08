---
title: "不 Pull Image 直接新增 Image Tag"
date: 2020-01-08T21:30:00+08:00
lastmod: 2020-01-08T21:30:31+08:00
draft: false
tags: ["Container","Docker","macOS"]
slug: "add-tag-without-pulling"
---

## 不 Pull Image 直接新增 Image Tag

最近發現整個 CI/CD 的流程耗時愈來愈長，雖說隨著愈來愈多的功能、測試，時間變長是可以預期的，但還是想要儘可能做些優化，同事在檢查 CI/CD flow 時發現 image retag step 耗時佔比較高，算是在不大改 CI/CD flow 前提中耗時最高的項目，於是列為優先調整項目，經過嘗試，確認可行，簡單紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.2
2. docker community 19.03.5
3. nexus 3.20.1

## 作法

這邊會透過自建的 docker registry 來示範，完整建立方式可以參考之前筆記 [使用 Docker 建立 Nexus3 的 Image Registry](https://blog.yowko.com/nexus-docker-image-rergistry), 以 redis 為例，目前只有 `5.0.3` 的 tag

![1onetag](https://user-images.githubusercontent.com/3851540/71984233-6a03b380-3263-11ea-8135-5134c4f06abb.png)

- script

    > 將下列 script 存為 `addtag.sh`(名稱可自取)

    ```bash
    # docker registry url
    REGISTRY_NAME="http://localhost:8082"
    # image name
    REPOSITORY=redis
    # 舊 image tag
    TAG_OLD=5.0.3
    # 想要增加的 image tag
    TAG_NEW=latest
    # 這是固定值
    CONTENT_TYPE="application/vnd.docker.distribution.manifest.v2+json"
    # 先取回 image 的 manifest
    MANIFEST=$(curl -v --user admin:pass.123 -H "Accept: ${CONTENT_TYPE}" "${REGISTRY_NAME}/v2/${REPOSITORY}/manifests/${TAG_OLD}")
    # 為 image 增加 tag (-v --user username:password 指定 docker registry 的驗證)
    curl -v --user admin:pass.123 -X PUT -H "Content-Type: ${CONTENT_TYPE}" -d "${MANIFEST}" "${REGISTRY_NAME}/v2/${REPOSITORY}/manifests/${TAG_NEW}"
    ```

- 執行

    - 雖然使用的是 `put` 方法，但所有 image tag 都會被留下，不是刪除舊 tag 加入新 tag
    - 執行速度很快，省下了整個 image 的下載時間，直接透過 http request 將需要的 tag 存入 docker registry 中

    ```bash
    sh addtag.sh
    ```

    ![3exeresult](https://user-images.githubusercontent.com/3851540/71984236-6a03b380-3263-11ea-82e0-5b7aa701481a.png)

    ![2twotag](https://user-images.githubusercontent.com/3851540/71984235-6a03b380-3263-11ea-9909-45ca78bbda64.png)

## 心得

最近遇到某些需求時，常讓我意識到自己做法逐漸改變中，過去我遇到自己覺得不符合使用技術正統做法的需求時，我會極力想要將需求導向可以透過正統做法解決的那個方向，但最近我會選擇先滿足需求，我想了想倒不是不再遵從正統做法，而是思考起會不會過去以為的正統做法其實只是因為當時的需求不夠充份或是設計有缺漏而造成現在無法透過當時定義的正統做法來解決問題

這次的例子也是，重覆為同一個 image 加上 tag，好像沒什麼必要，但我想可以實際解決問題、技術層面也原生支援，似乎也是個不錯解決方案

## 參考資訊

1. [How to Tag #Docker Images without Pulling them](https://dille.name/blog/2018/09/20/how-to-tag-docker-images-without-pulling-them/)
2. [Accessing and Using Your User Token](https://help.sonatype.com/repomanager3/security/security-setup-with-user-tokens)
3. [使用 Docker 建立 Nexus3 的 Image Registry](https://blog.yowko.com/nexus-docker-image-rergistry)
