---
title: "如何取得 Container 所使用的 Ip"
date: 2017-09-02T01:03:00+08:00
lastmod: 2017-09-02T01:03:45+08:00
draft: false
tags: ["docker","PowerShell"]
slug: "container-ip"
aliases:
    - /2017/09/container-ip.html
---
雖然一般來說都會將 container 的服務透過 port mapping 的動作，讓外部直接透過 docker host ip 來使用 container 所提供的服務，並不需要刻意取得 container 真正使用的 ip

不過為了確認 container 服務正確 container ip 就有功用了，我們可以在 docker host 透過使用 container ip 來驗證服務正確性

## 取得 Windows container 內部 ip

*   語法

    ```
    docker exec -it container_name_or_container_id ipconfig
    ```

*   範例

    ```
    docker exec -it 7f ipconfig
    ```

## 取得 Linux container 內部 ip

1.  方法一：直接針對 container 執行 ifconfig

    *   語法

        ```
        docker exec -it container_name_or_container_id ifconfig
        ```

    *   範例

        ```cmd
        docker exec -it oracle-xe-11g ifconfig
        ```

    ![1ifocnfig](https://user-images.githubusercontent.com/3851540/29979899-e4f07090-8f79-11e7-886b-2781d60dc3a1.png)

2.  方法二： container 未安裝 ifconfig 指令

    > 為了節省 docker image 的空間，很多非必要的套件都會被省略， ifconfig 就是其一，這時候就得透過其他方式來取得 ip

    ![2noifconfig](https://user-images.githubusercontent.com/3851540/29979900-e510c0c0-8f79-11e7-85be-46234233db41.png)

    *   語法 1

        ```cmd
        docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_container_id
        ```

        *   範例

            ```cmd
            docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jenkins
            ```

            ![3getip](https://user-images.githubusercontent.com/3851540/29979901-e51c00f2-8f79-11e7-9ea6-8be7a3c9d7a8.png)

    *   語法 2


        *   PowerShell 語法

            ```ps1
            docker inspect container_name_or_container_id | findstr '"IPAddress"'
            ```

        *   bash 語法

            ```bash
            docker inspect container_name_or_container_id | grep '"IPAddress"' | head -n 1
            ```

## 心得

docker 的使用與傳統 VM 的思維模式不同，常常會被以往的經驗所侷限而想要透過不合適的做法來解決 docker 遇到的問題，反而讓問題複雜化，不過這就是新技術導入時所需要克服與學習的

# 參考資訊

1.  [How to get IP address of running docker container](https://stackoverflow.com/questions/43692961/how-to-get-ip-address-of-running-docker-container)
2.  [10 Examples of how to get Docker Container IP Address](http://networkstatic.net/10-examples-of-how-to-get-docker-container-ip-address/)
