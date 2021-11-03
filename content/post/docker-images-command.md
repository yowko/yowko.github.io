---
title: "docker images 指令筆記"
date: 2019-09-08T01:30:00+08:00
lastmod: 2021-11-03T01:30:31+08:00
draft: false
tags: ["docker"]
slug: "docker-images-command"
---

## docker images 指令筆記

最近內部的 docker registry 在團隊努力地開發下，images 增長的速度相當地快速，不僅讓空間消耗快速，也讓整個 docker registry 顯得雜亂，於是我就在調整 docker registry 的 image 清除作業，在實際開發相關功能前花了不少時間建立本機的環境，避免對團隊的 docker registry 造成影響，過程中反覆要操作 image 相關指令，趁機筆記一下

## 基本環境說明

1. macOS Mojave 10.14.6
2. docker for mac 2.1.0.1 (37199)

## 關於 docker images 指令

- 指令用法

    ```bash
    docker images [OPTIONS] [REPOSITORY[:TAG]]
    ```

- 指令可用選項

    Name, shorthand|Description
    :---|:---
    --all , -a|顯示所有 images (不加這個選項，預設不會顯示中繼 images : repository 跟 tag 都為 `<none>`)
    --digests| 顯示 image 的 digests
    --filter , -f|依條件過濾 image
    --format|使用 Go template 來格式化 image 的顯示資訊
    --no-trunc|顯示完整 image 資訊
    --quiet , -q|只顯示 images id

1. `--all , -a`

    > 會顯示 repository 與 tag 為 `<none>` 的  image

    ![1docker-images-all](https://user-images.githubusercontent.com/3851540/64489669-6ac21400-d288-11e9-9bc2-37c5fd094b2d.png)

2. `--digests`

    > 增加顯示 `digest`，`digest` 是從 Docker 1.6.0 與 registry 2 開始加入用來識別 image 內容的 sha 值

    ![2digests](https://user-images.githubusercontent.com/3851540/64489670-6ac21400-d288-11e9-9104-2c83ef4e1a5b.png)

3. `--filter , -f`

    > 可用的 filter

    - dangling (boolean - true or false)

        > 用來過濾 images 是否有 tag (個人實測 - 只針對 tag : 只有 tag 為 none 會被列出，repository 為 none 的不會列出來)

        ```bash
        docker images -f dangling=true
        ```

        ![3filterdangling](https://user-images.githubusercontent.com/3851540/64489671-6ac21400-d288-11e9-9c72-775be0d61a2d.png)

        > 可使用下列語法直接刪除未被 tag 的 images

        ```bash
        docker rmi $(docker images -f "dangling=true" -q)
        ```

        ![4deletebydangling](https://user-images.githubusercontent.com/3851540/64489672-6b5aaa80-d288-11e9-9747-2544061199d1.png)

    - label

        - `label=<key>`

            > 具有指定 label 的 image

            ```bash
            docker images -f "label=org.label-schema.name"
            ```

            ![8filterlabel](https://user-images.githubusercontent.com/3851540/64489755-4d417a00-d289-11e9-8861-701ae18354ad.png)

        - `label=<key>=<value>`

            > 具有指定 label 且 value 為指定內容的 image

            ```bash
            docker images -f "label=org.label-schema.name=kafka"
            ```

            ![9filterlabelvalue](https://user-images.githubusercontent.com/3851540/64489756-4d417a00-d289-11e9-9042-999a1f663427.png)

    - before

        > 僅列出在指定的 `<image-name>[:<tag>]` , `<image id>` or `<image@digest>` 之前建立的 image

        ```bash
        $ docker images

        REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
        image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
        image2              latest              dea752e4e117        9 minutes ago        188.3 MB
        image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
        ```

        ```bash
        $ docker images --filter "before=image1"

        REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
        image2              latest              dea752e4e117        9 minutes ago        188.3 MB
        image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
        ```

    - since

         > 僅列出在指定的 `<image-name>[:<tag>]` , `<image id>` or `<image@digest>` 之後建立的 image

         ```bash
        $ docker images

        REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
        image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
        image2              latest              dea752e4e117        9 minutes ago        188.3 MB
        image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
        ```

        ```bash
        $ docker images --filter "since=image3"
        REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
        image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
        image2              latest              dea752e4e117        9 minutes ago        188.3 MB
        ```

    - reference

        > 依指定的 pattern 來過濾 image reference (這個選項依個人理解是對於 `image:tag` 的過濾)

        ```bash
        $ docker images

        REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
        busybox             latest              e02e811dd08f        5 weeks ago         1.09 MB
        busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
        busybox             musl                733eb3059dce        5 weeks ago         1.21 MB
        busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
        ```

        ```bash
        $ docker images --filter=reference='busy*:*libc'

        REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
        busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
        busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
        ```

        ```bash
        $ docker images --filter=reference='busy*:uclibc' --filter=reference='busy*:glibc'

        REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
        busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
        busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
        ```

4. `--format`

    > 透過 Go template 來格式化 docker images 的輸出內容

    可用變數|說明
    :---|:---
    .ID    |Image ID
    .Repository|Image repository
    .Tag|Image tag
    .Digest    |Image digest
    .CreatedSince|image 建立以來經過的時間
    .CreatedAt|image 建立時間
    .Size|image 大小

    ```bash
    docker images --format "{{.Repository}}:{{.Tag}}"| grep localhost:8082
    ```

    ![5format](https://user-images.githubusercontent.com/3851540/64489673-6b5aaa80-d288-11e9-8f50-8a934ece827a.png)

5. `--no-trunc`

    > 顯示完整 image 資訊

    - 未加 `--no-trunc`

        ![5trunc](https://user-images.githubusercontent.com/3851540/64489674-6b5aaa80-d288-11e9-8059-f89a12a4c602.png)

    - 加 `--no-trunc`

        ![6notrunc](https://user-images.githubusercontent.com/3851540/64489675-6bf34100-d288-11e9-82c6-4e1423582381.png)

6. `--quiet , -q`

    > 僅顯示 image id

    ![7quiet](https://user-images.githubusercontent.com/3851540/64489676-6bf34100-d288-11e9-8466-91278a919663.png)

## 心得

之前一直覺得 docker images 指令很單純，大概就是列出個 repository、tag、id，大部份情境下已經相當夠用，真正要用時才發現自己是井底之蛙，為了取得 repository:tag ，還一度想要用 xargs 指令手動處理，查完資料才發現原來 docker images 原生就提供便捷的做法，也才發現自己學藝不精，立馬動手紀錄加深印象

## 參考資訊

1. [docker images](https://docs.docker.com/engine/reference/commandline/images/)
2. [4.12 About Image Digests](https://docs.oracle.com/cd/E37670_01/E75728/html/ch04s12.html)
