---
title: "使用 docker 建立 CentOS 7 container"
date: 2017-05-06T01:00:00+08:00
lastmod: 2018-09-18T14:24:34+08:00
draft: false
tags: ["Docker","Linux"]
slug: "docker-centos-7-container"
aliases:
    - /2017/05/docker-centos-7-container.html
---
# 使用 docker 建立 CentOS 7 container
這是個奇特的情境：在 CentOS 7 實體機器上使用 docker 透過 CentOS 7 建立 CentOS 7 container ，聽來怪怪的XD，原因是 docker hub 上現有的 image 不符合公司使用需求，所以想要自行製作合適的 image，當然正統方式是使用 dockerfifle 來建立，但就目前我對 linux 語法熟悉度還做不到，只好用這招先解決問題，不過換個角度想：下個學習計劃就是 linux 了，真的永遠都學不完呀；

在建立 CentOS 7 container 的過程中有出現錯誤，所以紀錄一下

## docker hub 建議建立方式(無法建立成功)

*   建立語法

    > `docker run -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 local/c7-systemd-httpd`

*   錯誤訊息

    - 訊息內容

        ``` 
        [root@DK01 /]# docker run -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 local/c7-systemd-httpd
        Unable to find image 'local/c7-systemd-httpd:latest' locally
        Trying to pull repository docker.io/local/c7-systemd-httpd ... 
        Pulling repository docker.io/local/c7-systemd-httpd
        /usr/bin/docker-current: Error: image local/c7-systemd-httpd:latest not found.
        See '/usr/bin/docker-current run --help'.
        ```
    - 錯誤截圖
        
        ![1errormsg](https://cloud.githubusercontent.com/assets/3851540/25735814/830191b6-31a0-11e7-8708-74ccdcf0c931.png)

## 正確建立方式 (擇一即可)

1.  建立語法 一(由 docker hub 建議語法改善而來)

    > `docker run --privileged --name centos7 -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 -d renizgo/c7-systemd-httpd /usr/sbin/init`

2.  建立語法 二(語法較直覺)

    > `docker run --privileged --name centos7 -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 -d centos /usr/sbin/init`

## 心得

docker 還在快速發展，語法常有異動，錯誤的解決方式也經常有變化，使用時需要多留意；

另外再次提醒大家：客製 image 正統作法是透過 dockerfile 來 build 出適合的 image，不是將 container 裝上所需工具後再 commit 為 image

# 參考資訊

1.  [The official build of CentOS](https://hub.docker.com/_/centos/)
