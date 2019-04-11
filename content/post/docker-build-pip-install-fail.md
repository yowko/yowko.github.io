---
title: "解決 Docker build pip install fail"
date: 2018-06-16T12:27:00+08:00
lastmod: 2018-10-07T12:27:41+08:00
draft: false
tags: ["Docker","Linux","Python"]
slug: "docker-build-pip-install-fail"
aliases:
    - /2018/06/docker-build-pip-install-fail.html
---
# 解決 Docker build pip install fail
近期工作之餘將部份時間花在學習 Kubernetes 上，過程中嘗試透過 Dockerfile 建立 pyhon application 的 image 來部署至 Kubernetes 中打算用來做一些實際應用情境的演練，原以為會卡在 Kubernetes 的網路設定上，想不到我太過樂觀，連第一步的 Dockerfile 都 build fail，如果沒有 image 更不用提建立 container 及部署至 Kubernetes 

因為 Kubernetes 相關知識尚未上手，非常容易踩雷碰壁，為此我還重建了好幾次 VM 來反覆驗證測試，過程中也遇到好幾次 Dockerfile pip install fail 的狀況，每次都要重新查設定方式，於是就來筆記一下囉

## 錯誤訊息
- 訊息內容
    
    ```
    Collecting Flask
    Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.VerifiedHTTPSConnection object at 0x7f21e335f128>: Failed to establish a new connection: [Errno -3] Temporary failure in name resolution',)': /simple/flask/
    ```
- 錯誤截圖
    
    ![1error](https://user-images.githubusercontent.com/3851540/41495678-3cb54694-715f-11e8-82f8-38aeafbb81e0.png)    

## 問題原因

> Docker 未使用正確的 DNS 設定，造成解析錯誤

## 解決方式

> 指定 Docker 使用的 DNS

1. 開啟 `/etc/default/docker`
    
    ```
    nano /etc/default/docker
    ``` 
2. 加入 DNS 設定
    
    ```
    DOCKER_OPTS="--dns 8.8.8.8"
    ```
    
    ![2dockerdns](https://user-images.githubusercontent.com/3851540/41495679-3cf8e264-715f-11e8-9fe9-de3cf5e8e4d3.png)
3. 重啟 Docker
    
    ```
    systemctl restart docker
    ```
4. 成功 build Dockerfile
    
    ![3success](https://user-images.githubusercontent.com/3851540/41495680-3d27018a-715f-11e8-8325-c019eb5ddd28.png)

* [Can't install pip packages inside a docker container with Ubuntu](https://stackoverflow.com/questions/28668180/cant-install-pip-packages-inside-a-docker-container-with-ubuntu) 有提供其他解決方式，如果上述做法無法解決可以試試其他方法

## 心得
前幾天跟同事討論到 Dokcer 實際使用上的心得：感覺上 Docker 遇到的坑不少，常常遇到問題，雖然不是解決不了但常常都得靠眾多網友的智慧，官方不一定有明確的建議，另外常常不同版本間的解決方式也不太一樣：可能前一版能用的做法新版就有其他方法。不過我覺得這就是 Docker 相關應用仍在快速發展的表現，雖然功能上還不穩定，但需要的功能可能在短期內就可以被實現，後續表現還是相當值得期待

# 參考資訊
1. [Can't install pip packages inside a docker container with Ubuntu](https://stackoverflow.com/questions/28668180/cant-install-pip-packages-inside-a-docker-container-with-ubuntu)