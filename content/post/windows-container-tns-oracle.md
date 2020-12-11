---
title: "在 Windows Container 使用 tns 連線 Oracle"
date: 2017-11-22T00:07:00+08:00
lastmod: 2020-12-11T00:07:19+08:00
draft: false
tags: ["Docker","Oracle","Windows 10","Windows Server 2016"]
slug: "windows-container-tns-oracle"
aliases:
    - /2017/11/windows-container-tns-oracle.html
---
# 在 Windows Container 使用 tns 連線 Oracle
之前筆記 [不用安裝 Oracle Client 使用 C# 透過 tnsnamses.ora 連結 Oracle](/2017/11/c-sharp-oracle-tns-without-client.html) 介紹到如何讓 server 不用安裝 Oracle Client 就可以使用 tns 存取 Oracle，其實最終目的就是想要在 Windows Container 透過 tns 來連線 Oracle，主要原因就是目前公司系統大多數使用 tns 來連線 Oracle，一來不想因為測試來修改連線方式，二來也不可能完全放棄原本做法，另外還有個重要的原因是 Oracle Clinet 非常不好安裝

立馬來看看 Windows Container 如何使用 TNS 連線 Oracle 吧

## 準備 Windows Container 執行的程式

*   將預計運行在 Windows Container 中的程式先打包
*   記得設定程式連線字串 (connection string) 使用 tns


## 準備 tnsnames.ora

*   將連線相關資訊以 `tnsnames.ora` 格式儲存

    ```
    yowkooracle =
    (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.10.99)(PORT = 1521))
        (CONNECT_DATA =
        (SERVICE_NAME = xe)
        )
    )
    ```

## 準備 dockerfile

```dockerfile
#指定基礎 os image
FROM microsoft/aspnet

# image 維護者資訊
MAINTAINER yowko@yowko.com

#將網站複製到 container 中
COPY ./PublishOutput/ /inetpub/wwwroot

#建立放置 tnsnames.ora 的資料夾
RUN mkdir oracle

#將 tnsnames.ora 複製至上面建立的資料夾中
COPY ./tnsnames.ora /oracle

#設定環境變數 TNS_Admin 指向上面存放 tnsnames.ora 的資料夾
ENV TNS_Admin "C:\oracle"

#對外使用 80 port
EXPOSE 80

# 預設執行動作，可用來避免 container 自動停止
CMD [ "ping localhost -t" ]
```

*   專案結構示意
    *   PublishOutput

        > 欲部署至 container 的網站內容

    *   tnsnames.ora
    *   dockerfile

    ![1folder](https://user-images.githubusercontent.com/3851540/33082375-978540dc-cf17-11e7-8411-c233bb61b030.png)

*   建立 image
    *   語法

        ```
        docker build -t {image tag} {dockerfile 所在位置}
        ```

    *   範例

        ```
        docker build -t mvcwebsitewithtns:v1 c:\mvcwebsite
        ```

    ![2buildimage](https://user-images.githubusercontent.com/3851540/33082376-97ad0a04-cf17-11e7-87eb-74355e4f0afb.png)

*   建立 container
    *   語法

        ```
        docker run -d -p {port}:80 --name {container name} {image tag}
        ```
    *   範例

        ```
        docker run -d -p 80:80 --name mvcwebsite mvcwebsitewithtns:v1
        ```
    
    ![3createcontainer](https://user-images.githubusercontent.com/3851540/33082377-97d7043a-cf17-11e7-958d-bd6fcd533a7b.png)

*   成功連線

    ![4result](https://user-images.githubusercontent.com/3851540/33082379-97fec97a-cf17-11e7-9e6e-407da67b57e2.png)

## 心得

Windows 使用 tns 連線 Oracle 的問題卡了好幾天，表面上主要原因是不想安裝 Oracle Client，而實際上是因為想裝也裝不太起來 XD，Oracle Client 在有 GUI 跟前人文件都不一定能裝起來了，要 silent install 不知道要試多久才會成功，幸虧有先搞定 tns 的問題才讓 Windows Container 使用 tns 連線 Oracle 沒有遇到多大障礙

但也不是完全沒有障礙，我一開始使用 Windows 10 來測試 Windows Container，但老是無法成功，瀕臨放棄邊緣時改用 Windows Server 2016 馬上就解決問題，至今在 Windows 10 上仍無法在 container 中透過 tns 連線 Oracle (原因`個人推測`是 hyper-v container 多隔了一層的關係)，除此之外我也發現 Windows 10 不支援 `docker cp` 指令

# 參考資訊

1.  [Cannot update PATH variable in a windows docker container](https://forums.docker.com/t/cannot-update-path-variable-in-a-windows-docker-container/30960)
2.  [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
3.  [Windows 上的 Dockerfile](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile?WT.mc_id=DOP-MVP-5002594)
4.  [Migrating ASP.NET MVC Applications to Windows Containers](https://docs.microsoft.com/en-us/aspnet/mvc/overview/deployment/docker-aspnetmvc?WT.mc_id=DOP-MVP-5002594)
5.  [不用安裝 Oracle Client 使用 C# 透過 tnsnamses.ora 連結 Oracle](/2017/11/c-sharp-oracle-tns-without-client.html)
