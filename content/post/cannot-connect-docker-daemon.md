---
title: "CentOS 7 Docker 無法連線 Docker daemon？！"
date: 2018-06-06T04:00:00+08:00
lastmod: 2018-10-07T10:43:18+08:00
draft: false
tags: ["Docker","Linux","VirtualBox"]
slug: "cannot-connect-docker-daemon"
aliases:
    - /2018/06/cannot-connect-docker-daemon.html
---
# CentOS 7 Docker 無法連線 Docker daemon？！
同事想在 VirtualBox 上建立 CentOS 7 虛擬環境，並在 VM 中使用 Docker，在安裝 Docker 時一如往常相當順利直到實際建立 container 時卻遇到無法連線：`Cannot connect to the Docker daemon at unix:///var/run/docker.sock` 的提示訊息，重啟 Docker service 時也失敗並得到新的錯誤：`Job for docker.service failed because the control process exited with error code.`

解決這個問題的過程中學到以前不知道的內容與設定，順手紀錄一下備忘

## 錯誤訊息
1. 訊息內容
    
    ```
    [root@localhost ~]# docker version                                                 
    Client:               
    Version:         1.13.1                                         
    API version:     1.26                         
    Package version:                           
    Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
    ```
2. 錯誤截圖
    
    ![1error](https://user-images.githubusercontent.com/3851540/41012347-b7624782-6973-11e8-8a74-cced8daa1b14.png)

## 嘗試重新啟動 Docker
1. 重新啟動指令
    
    ```
    sudo systemctl restart docker
    ``` 
2. 重啟失敗
    - 失敗訊息
        
        ```
        Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details
        ```  
    - 失敗截圖 
        
        ![2restartfail](https://user-images.githubusercontent.com/3851540/41012348-b78c721e-6973-11e8-828a-704324b785ba.png)

## 詳細錯誤
1. `systemctl status docker.service`
    - 訊息內容
        
        ```
        ● docker.service - Docker Application Container Engine
        Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
        Active: failed (Result: exit-code) since Wed 2018-06-06 01:00:01 CST; 21min ago
            Docs: http://docs.docker.com
        Process: 3508 ExecStart=/usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current --init-path=/usr/libexec/docker/docker-init-current --seccomp-profile=/etc/docker/seccomp.json $OPTIONS $DOCKER_STORAGE_OPTIONS $DOCKER_NETWORK_OPTIONS $ADD_REGISTRY $BLOCK_REGISTRY $INSECURE_REGISTRY $REGISTRIES (code=exited, status=1/FAILURE)
        Main PID: 3508 (code=exited, status=1/FAILURE)

        Jun 06 01:00:00 localhost.localdomain systemd[1]: Starting Docker Application Container Engine...
        Jun 06 01:00:00 localhost.localdomain dockerd-current[3508]: time="2018-06-06T01:00:00.466231528+08:00" level=info msg="libcontainerd: new contai...: 3515"
        Jun 06 01:00:01 localhost.localdomain dockerd-current[3508]: Error starting daemon: SELinux is not supported with the overlay2 graph driver on th...=false)
        Jun 06 01:00:01 localhost.localdomain systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
        Jun 06 01:00:01 localhost.localdomain systemd[1]: Failed to start Docker Application Container Engine.
        Jun 06 01:00:01 localhost.localdomain systemd[1]: Unit docker.service entered failed state.
        Jun 06 01:00:01 localhost.localdomain systemd[1]: docker.service failed.
        Hint: Some lines were ellipsized, use -l to show in full.
        ``` 
    -  訊息截圖
        
        ![3servicestatus](https://user-images.githubusercontent.com/3851540/41012349-b7b72aa4-6973-11e8-9c4e-3cbef5508328.png)
2. `journalctl -xe`
    
    > 錯誤內容非常多，就不另外貼出來了

    ![4jx](https://user-images.githubusercontent.com/3851540/41012350-b7e1ffd6-6973-11e8-9444-03f50a5b19a0.png)

## 問題發生原因
透過檢視 `journalctl -xe` 的錯誤資訊，可以發現重要線索：`linux kernel 中的 SELinux 不支援 overlay2 graph driver`

>![5rootcause](https://user-images.githubusercontent.com/3851540/41012351-b80bb196-6973-11e8-945e-a8dd14516d8b.png)

## 解決方式(擇一即可)
1. 自行啟動 Docker
    - 一個 process (一個 linux 連線) 執行下列指令
        - `nohup` 登出不中斷執行
        - Docker 預設是透過 `unix:///var/run/docker.sock` 執行
        - 可以透過 `-H` 加上 tcp 即可允許 curl 連線操作
            
            > `nohup docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock`
        
        ```
        nohup docker daemon -H unix:///var/run/docker.sock
        ``` 
    - 另一個 proccess (另一個 linux 連線) 就可以正常使用 docker 指令
2. 修改 Docker service 啟動指令
    - 開啟 `/usr/lib/systemd/system/docker.service` 
    - 修改 `ExecStart`
        
        ```
        ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock
        ``` 
    - 重新讀取設定
        
        ```
        systemctl daemon-reload
        ``` 
    - 重啟 Docker service
        
        ```
        systemctl restart docker.service
        ``` 
3. 停用 Docker 的 selinux
    - 開啟 `vi /etc/sysconfig/docker`
    - 將 `--selinux-enabled` 改為 `--selinux-enabled=false`
        
        ![6selinux](https://user-images.githubusercontent.com/3851540/41012345-b70bba66-6973-11e8-931f-e74bd6523a8d.png)

        
        ![7selinux](https://user-images.githubusercontent.com/3851540/41012346-b7385b0c-6973-11e8-90b5-53ee1aea6f64.png)
    - 啟動 Docker service
        
        ```
        systemctl start docker.service
        ``` 

## 心得
雖然還沒確定問題發生原因前就猜測是設定造成的，但透過 `setenforce 0` 暫時關閉 SElinux 後仍無法正常使用，原以為已可排除 SElinux 設定問題，想不到只是自己才疏學淺呀。

經過同事這次遇到的狀況，才得以多瞭解到原來 Docker service 預設是使用 `unix:///var/run/docker.sock` 進行服務，不僅可以手動建立 Docker daemon 還可以透過 tcp 綁定啟用 Docker Remote API

近幾個月 Kubenetes 的網路聲量很大，相較之下 Docker 近期發展上比較少大型爆發性變動，正面思考是相關 API 已經穩定可以符合多數情境了，只是就我自己的使用經驗還有太多東西不知道，常常遇到問題要花不少時間才能解決，這樣跨入 Kubenetes 難免多了一些不安定因素，但 Kubenetes 的學習不能等待呀  哈哈

# 參考資訊
1. [Cannot connect to the Docker daemon at unix:/var/run/docker.sock. Is the docker daemon running?](https://stackoverflow.com/questions/44678725/cannot-connect-to-the-docker-daemon-at-unix-var-run-docker-sock-is-the-docker)
2. [Docker Remote API 如何使用？](https://www.zhihu.com/question/24852884)
3. [ocker啟動報錯：Error starting daemon: SELinux is not supported with the overlay2 graph driver on this ke](https://blog.csdn.net/a1010256340/article/details/80106156)