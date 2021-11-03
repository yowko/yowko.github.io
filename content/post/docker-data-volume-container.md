---
title: "Docker 如何在多個 Container 間共享資料"
date: 2017-05-11T23:08:00+08:00
lastmod: 2021-11-03T11:49:16+08:00
draft: false
tags: ["Docker","Linux"]
slug: "docker-data-volume-container"
aliases:
    - /2017/05/docker-data-volume-container.html
---
## Docker 如何在多個 Container 間共享資料

最近跟公司同事一起將部份公司服務改使用 docker 來架設，為了確保相關資料及設定不會隨著 container 消滅而不見，所以一定需要將資料保存在 container 外部，畢竟 container 的生命周期一般都是非常短暫的，關於這點 docker 已經有 volume 的設計，只是一旦你需要指定的 volume 數量很多或是需要建立很多個相同的 container 時，你一定會想要找到更方便的方式，而這個方法就是：`Data Volume Container`，其實就是個一般容器，特別的是你可以將許許多多的 volume 指定在 `Data Volume Container` 上，再將其他需要共同資料的 container 指向這個 data volume container 即可，一來大量指定 volume 的動作只需要在 data volume container 設定一次就好，二來如果需要修改也只需要改 data volume container 的對應即可，再來 data volume container 也支援備份及還原的功能，功能比較全面，立馬來看看該如何設定吧

## 建立 Data Volume Container

* data volume container 不需常駐執行(docker ps -a 查看，可以是處於 `Exited`)
* image 可以找個輕量 image 即可
* 建立語法：

    ```bash
    sudo docker run -d -p 9122:22 -v /storage/config:/gitlab/etc/ -v /storage/logs:/gitlab/var/log/ -v /storage/data:/gitlab/var/data --name storage centos
    ```

## 一般 Container 使用 Data Volume Container

* 使用 `--volumes-from` 來掛載 Data Volume Container
* 掛載語法

    ```bash
    sudo docker run -d -p 443:443 -p 80:80 -p 22:22 --restart always --volumes-from storage --name gitlabce gitlab/gitlab-ce:latest
    ```

## 備份及還原 Data Volume Container

* 備份

  * 說明

    > 將 data volume container (`--volumes-from storage`) 掛載在一個用完即刪 (`--rm`) 的 centos container 上，並將 container 中的 backup 資料夾對應至當前目錄 (`-v $(pwd):/backup`) ，最後針對這個 centos container 執行`tar cvf /backup/backup.tar /gitlab`(將 `/gitlab` 資料夾封裝至 `/backup/backup.tar`)

  * 語法

    ```bash
    docker run --rm --volumes-from storage -v $(pwd):/backup centos tar cvf /backup/backup.tar /gitlab
    ```

* 還原

  * 說明

    > 建立新的 data volume container，將資料還原至這個新建的 data volume container 中

  * 語法

    ```bash
    docker run -v /gitlab --name storage2 centos

    docker run --rm --volumes-from storage2 -v $(pwd):/backup centos bash -c "cd /gitlab && tar xvf /backup/backup.tar --strip 1"
    ```

## 參考資料

1. [Docker - Data Volume Container](https://peihsinsu.gitbooks.io/docker-note-book/content/docker_data_volume_container.html)
2. [Docker Practice - Volume Management](https://puremonkey2010.blogspot.tw/2015/05/docker-practice-volume-management.html)
3. [Manage data in containers](https://docs.docker.com/engine/tutorials/dockervolumes/)
