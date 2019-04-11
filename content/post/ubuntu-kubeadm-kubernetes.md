---
title: "Windows 慣用者如何在 Ubuntu Server 16.04 LTS 上使用 kubeadm 架設 Kubernetes (K8s)"
date: 2018-05-14T02:15:00+08:00
lastmod: 2018-10-06T02:15:30+08:00
draft: false
tags: ["docker","Kubernetes"]
slug: "ubuntu-kubeadm-kubernetes"
aliases:
    - /2018/05/ubuntu-kubeadm-kubernetes.html
---
# Windows 慣用者如何在 Ubuntu Server 16.04 LTS 上使用 kubeadm 架設 Kubernetes (K8s)
原本是想將 Kubernetes 架設在 Ubuntu Server 17.10 上，但需要透過一些小手法才能順利執行 container，為了避免忘記如何架設，決定先退而求其次的使用 Ubuntu Server 16.04 LTS 紀錄一下先，網路上有許多文章在介紹如何安裝 Kubernetes，我相信我不會寫得比網路上的大大來得好，但我想站在一個 Windows 環境慣用者的角度來紀錄該如何使用 Ubuntu 及安裝 Kubernetes，主要就是因為我自己在實際安裝時就遇到不少操作上問題XD

Kubernetes - K8s 是用於自動部署、擴展和管理容器化（containerized）應用程式的開源系統，前身為 Borg (節錄自 [Kubernetes - 維基百科，自由的百科全書 - Wikipedia](https://zh.wikipedia.org/wiki/Kubernetes))

近來 Kubernetes 漸漸有擺脫其他 Orchestration 工具(像是 docker swarm,Mesos... )的態勢，功能發展及業界關注度明顯較為熱絡，雖然一直相當看好 docker 的後續發展但也仍在觀望到底該選擇什麼樣的 Orchestration tool，近來覺得應該可以先試試 Kubernetes ，就來看看該如何在 Ubuntu Server 16.04 LTS 上架設 Kubernetes 吧

## 開始安裝前的小提醒
1. 指令常需要以 `root` (類似 Windows 的系統管理員角色) 執行
    
    > 在每個指令前加上 `sudo` 來執行
2. 切換成 `root` 使用者
    - 切換前需先為 `root` 設定密碼，否則切換時會一直遇到認證失敗 (Authentication failure)
        
        > 使用  `sudo passwd` 設定密碼
    - 實際切換為 `root`
        
        ```
        su root
        ``` 
        或
        
        ```
        su
        ```

## 安裝 Docker

> 記得執行指令前加上 `sudo` 或是切換為 `root`

1. 更新套件
    
    > apt-get 是 Ubuntu 內建的應用程式管理工具，作用等同於 Windows 的 Chocolatey ，只是在 Windows 上 Chocolatey 需要自行安裝
    
    ```
    apt-get update
    ``` 
2. 直接從 Ubuntu repository 安裝 Docker
    
    ```
    apt-get install -y docker.io
    ```
3. 確認安裝成功
    
    ```
    docker version
    ``` 
    >![1installdocker](https://user-images.githubusercontent.com/3851540/39970266-f6afaa12-571a-11e8-8052-b12bb6dd604e.png)

## 安裝 Kubernetes

> 記得執行指令前加上 `sudo` 或是切換為 `root`

1. 更新套件
    
    ```
    apt-get update
    ``` 
2. 安裝讓 apt 支援 ssl 的套件 ` apt-transport-https`
    
    ```
    apt-get  install  -y  apt-transport-https
    ```
3. 從遠端下載 key 並加入 APT 信任清單中
    
    ```
    curl  -s https://packages.cloud.google.com/apt/doc/apt-key.gpg |  apt-key  add  - 
    ```
    
    ![2aptkeyadd](https://user-images.githubusercontent.com/3851540/39970267-f6d975f4-571a-11e8-93fd-4c4af688b8a0.png)
4. 將 apt 來源位置覆蓋至 kubernetes.list
    
    > 這邊要留意一下從 Windows 貼上因為換行符號與 linux 不同，可能會有問題 
    
    ```
    cat  <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial  main 
    EOF
    ```
    
    ![3catlist](https://user-images.githubusercontent.com/3851540/39970268-f701c644-571a-11e8-9951-e3b8324b317d.png)
5. 更新套件
    
    ```
    apt-get update
    ``` 
6. 安裝 `kubelet`，`kubeadm`，`kubectl`
    - kubelet
        
        > 安裝於 cluster 所有機器上的元件，用來啟動 pod 跟 container 
    - kubeadm
        
        >是 Kubernetes 官方提供的快速安裝和初始化 Kubernetes cluster 的工具 
    - kubectl
        
        > Kubernetes 的命令列工具
    
    ```
    apt-get install -y kubelet kubeadm kubectl
    ``` 
7. 確認安裝成功
    
    ```
    kubeadm version
    ``` 
    
    ![4kubeadmversion](https://user-images.githubusercontent.com/3851540/39970269-f7345668-571a-11e8-9c8d-cd053cd06894.png)

## 關閉系統 Swap

>Swap 類似 Windows 中的虛擬記憶體，從 Kubernetes 1.8 開始要求關閉系統的 Swap

- 關閉 Swap
    
    ```
    swapoff -a
    ```
- 修改 `/etc/fstab`，避免 `Swap` 自動掛載
    
    ```
    sed -e '/swap/ s/^#*/#/' -i /etc/fstab
    ``` 
- 確認關閉
    
    ```
    free -m
    ```
    
    ![5swapoff](https://user-images.githubusercontent.com/3851540/39970270-f76661c6-571a-11e8-857f-7b8b8161395a.png)

## 建立 cluster
1. 初始化 master 
    
    > `--pod-network-cidr` 是為後面的 Pod 網路套件 (`flannel`) 做準備
    
    ```
    kubeadm init --pod-network-cidr=10.244.0.0/16
    ```
    
    ![6initsuccess](https://user-images.githubusercontent.com/3851540/39970271-f79193f0-571a-11e8-8f4e-f941bd6f49e4.png)
2. 確保有權限執行 `kubectl `
    - 非 root user
        
        ```
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ``` 
    - root user 
        
        ```
        export KUBECONFIG=/etc/kubernetes/admin.conf
        ``` 
3. 啟用 bash 自動完成 
    
    ```
    apt-get install -y bash-completion
    echo "source /etc/bash_completion" >> ~/.bashrc
    echo "source <(kubectl completion bash)" >> ~/.bashrc
    source <(kubectl completion bash)

    ``` 
4. 安裝 Pod 網路套件
    
    > 這邊以 `flannel` 為例
     
    ```
    kubectl apply --namespace kube-system -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ```
    
    ![7flannel](https://user-images.githubusercontent.com/3851540/39970272-f7bcc390-571a-11e8-816c-e5fa0313ca9e.png)

## 加入其他 Node 
1. 新的 Node 上需先完成前面的 `安裝 Docker`，`安裝 Kubernetes`，`關閉系統 Swap`
2. 執行 join 指令
    
    > 指令內容會在 master 執行 `kubeadm init` 時產生
    
    ```
    kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
    ``` 
    
    ![8joinscript](https://user-images.githubusercontent.com/3851540/39970273-f80b14b4-571a-11e8-9f28-9ba0d998d34b.png)
3. 加入成功
    
    ![9nodejoined](https://user-images.githubusercontent.com/3851540/39970327-0f6ba410-571c-11e8-85fa-7526f2b8ae03.png)
4. 至 master 上確認
    
    ```
    kubectl get nodes
    ``` 
    
    ![10getnodes](https://user-images.githubusercontent.com/3851540/39970275-f8604600-571a-11e8-9508-971b428d8f29.png) 

## 心得
Kubernetes 整個架構設計相當具有擴充性，也因為如此讓 Kubernetes 變得複雜，在學習上有不小的門檻，加上我本身平常使用的是 Windows 環境，常常卡在 linux 相關設定或是語法上，但也是透過這樣的機會才能試著學習 linux 上的操作以及觀念

雖然今天初步完成了 Kubernetes cluster 的架設，不過許多設定都還不清楚，甚至名詞的掌握度也不高，除此之外 Windows node 的介接或是實際的部署都是挑戰呀，希望接下來有機會可以實際應用在工作上

# 參考資訊
1. [為何Orchestration是企業擁抱容器的關鍵？](https://www.ithome.com.tw/news/108746)
2. [What is the meaning of combined commands `curl` + `apt-key add`?](https://askubuntu.com/questions/900693/what-is-the-meaning-of-combined-commands-curl-apt-key-add)
3. [章 6. 維護與更新：APT 工具](https://debian-handbook.info/browse/zh-TW/stable/apt.html)
4. [Play with Kubernetes](https://labs.play-with-k8s.com/)
5. [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
6. [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
