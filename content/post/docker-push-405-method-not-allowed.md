---
title: "Docker Push 出現 405 Method Not Allowed 錯誤？！"
date: 2018-06-16T19:39:00+08:00
lastmod: 2018-10-07T19:39:54+08:00
draft: false
tags: ["Docker","Linux"]
slug: "docker-push-405-method-not-allowed"
aliases:
    - /2018/06/docker-push-405-method-not-allowed.html
---
# Docker Push 出現 405 Method Not Allowed 錯誤？！
之前筆記 [解決 Docker build pip install fail](https://blog.yowko.com/2018/06/docker-build-pip-install-fail.html) 提到在練習 Kubernetes 過程中執行 docker build 指令時會出現 pip install fail 的 error，問題發生原因是 docker 的 DNS 解析不正確造成的，解決方式是設定 Docker 的 DNS。

完成 docker build 產生的 image 只會存在執行指令的機器上，除非手動打包 image 至其他 Kubernetes Node 上，否則在執行 Kubernetes 部署時會出現找不到 image 而出現部署失敗的錯誤訊息，但 Kubernetes Node 可以動態增減，手動複製並沒有真正解決問題，透過將 image push 至 registry 才可以一勞永逸，只是想不到一個 `docker push` 也讓我卡關了XD  就來看看問題發生原因及解決方式吧

## 環境說明
1. linux 資料
    
    >`hostnamectl` 
    
    ```
    Static hostname: k8s06141
    Icon name: computer-vm
    Chassis: vm
    Machine ID: a34d496ad3514c718a709b8443ab1962
    Boot ID: 018da92531e14fdbb73d1bcad59731a3
    Virtualization: microsoft
    Operating System: Red Hat Enterprise Linux Server 7.5 (Maipo)
    CPE OS Name: cpe:/o:redhat:enterprise_linux:7.5:GA:server
    Kernel: Linux 3.10.0-862.2.3.el7.x86_64
    Architecture: x86-64
    ``` 
3. docker version
    
    ```
    Client:
    Version:         1.13.1
    API version:     1.26
    Package version: docker-1.13.1-63.git94f4240.el7.x86_64
    Go version:      go1.9.2
    Git commit:      94f4240/1.13.1
    Built:           Mon Apr 30 15:45:42 2018
    OS/Arch:         linux/amd64

    Server:
    Version:         1.13.1
    API version:     1.26 (minimum version 1.12)
    Package version: docker-1.13.1-63.git94f4240.el7.x86_64
    Go version:      go1.9.2
    Git commit:      94f4240/1.13.1
    Built:           Mon Apr 30 15:45:42 2018
    OS/Arch:         linux/amd64
    Experimental:    false
    ```
## 錯誤訊息
1. 訊息內容
    
    ```
    error parsing HTTP 405 response body: invalid character '<' looking for beginning of value: "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 3.2 Final//EN\">\n<title>405 Method Not Allowed</title>\n<h1>Method Not Allowed</h1>\n<p>The method POST is not allowed for the requested URL.</p>\n"
    ``` 
3. 錯誤截圖
    
    ![1error](https://user-images.githubusercontent.com/3851540/41497900-ff92a58a-7191-11e8-91a6-92efa6d3fd1a.png) 

## 問題發生原因

```
- the original BZ wanted the ability to __docker search__ the RH registry for images (nothing to do with push/pull) 
- the original BZ resolution added the rhel registry as the default one, making pushing images just go to the RH registry (which is plain wrong to me, both in RHEL and Fedora)
```

> 原始計畫想要可以使用 `docker search` 指令來搜尋在 redhat registry 上的 image (只搜尋沒有 `pull` 及 `push` 的需求 )，後來決定將 redhat registry 設為預設 registry，也就造成 image push 至 redhat registry 中


## 解決方式

> 官方 Bugzilla 顯示問題在 `docker-1.13.1-6.git5be1549.fc26` 已被解決，只是我個人仍遇到問題，以下提供個人測試可用解決方式

1. 開啟 `/etc/containers/registries.conf`
    
    ```
    nano /etc/containers/registries.conf
    ```
2. 在 `registries` 中加入 `docker.io` 並排在第一順位
    - 原設定
        
        ```
        [registries.search]
        registries = ['registry.access.redhat.com']
        ``` 
    - 新設定
        
        ```
        [registries.search]
        registries = ['docker.io','registry.access.redhat.com']
        ```
    >![2setting](https://user-images.githubusercontent.com/3851540/41497901-ffc0889c-7191-11e8-9284-f64fe926eb62.png)
3. 重新啟動 Docker
    
    ```
    systemctl restart docker
    ```
4. Docker push 成功
    
    ![3success](https://user-images.githubusercontent.com/3851540/41497902-ffea2fda-7191-11e8-8479-49fa4b2416c6.png)

## 心得
原以為輕鬆寫意的動作，竟然連撞兩次牆，看來 Kubernetes 之路不好走呀

不過據 Red Hat Bugzilla 的資料 [Bug 1434897 - docker tries to push to registry.redhat.comr](https://bugzilla.redhat.com/show_bug.cgi?id=1434897) 看來，官方已判定為 bug 並完成修正，待版本更新後問題應該就不會再發生了，但在那之前筆記一下可以節省重建環境時遇到相同狀況都要重新查語法的時間，把時間留給 Kubernetes 的核心功能比較實在呀 @@"

# 參考資訊
1. [Bug 1434897 - docker tries to push to registry.redhat.comr](https://bugzilla.redhat.com/show_bug.cgi?id=1434897)