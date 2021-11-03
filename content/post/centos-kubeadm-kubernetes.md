---
title: "Windows 慣用者如何在 Red Hat Enterprise Linux 7.5 (CentOS ) 上使用 kubeadm 架設 Kubernetes (K8s)"
date: 2018-05-20T10:16:00+08:00
lastmod: 2021-11-03T20:30:50+08:00
draft: false
tags: ["Kubernetes","Linux"]
slug: "centos-kubeadm-kubernetes"
aliases:
    - /2018/05/centos-kubeadm-kubernetes.html
---
## Windows 慣用者如何在 Red Hat Enterprise Linux 7.5 (CentOS ) 上使用 kubeadm 架設 Kubernetes (K8s)

之前筆記 [Windows 慣用者如何在 Ubuntu Server 16.04 LTS 上使用 kubeadm 架設 Kubernetes (K8s)](/ubuntu-kubeadm-kubernetes) 紀錄到如何在 Ubuntu 上使用 kubeadm 建立 Kubernetes cluster，考量到公司使用的 linux 都為 CentOS (RHEL) 所以趁著印象還新鮮時趕緊紀錄一下在 CentOS 上架設 Kubernetes 的做法

## 開始安裝前的小提醒

1. 指令常需要以 `root` (類似 Windows 的系統管理員角色) 執行

    > 在每個指令前加上 `sudo` 來執行
2. 切換成 `root` 使用者
    - 切換前需先為 `root` 設定密碼，否則切換時會一直遇到認證失敗 (Authentication failure)

        > 使用  `sudo passwd` 設定密碼
    - 實際切換為 `root`

        ```bash
        su root
        ```

        或

        ```bash
        su
        ```

## 安裝 Docker

> 記得執行指令前加上 `sudo` 或是切換為 `root`

1. 直接從 OS 中內建的 CentOS-Extras 庫安裝 Docker

    > yum（ Yellow dog Updater, Modified）是一個在 Fedora 和 RedHat 以及 SUSE 中基於 RPM 檔的軟體管理工具，能夠從指定的伺服器自動下載 RPM 檔並且安裝，可以處理依賴性關係，並且一次安裝所有相依賴的軟體，無須自行下載、安裝，作用等同於 Windows 的 Chocolatey ，只是在 Windows 上 Chocolatey 需要自行安裝

    ```bash
    yum install -y docker
    ```

    - `-y`：安裝過程選項提示皆預設選擇 "yes"
2. 設定 Docker 在系統開機時自動啟動

    ```bash
    systemctl enable docker
    ```

    ![1enabledocker](https://user-images.githubusercontent.com/3851540/40274856-1d496854-5c14-11e8-857e-11acae4b93ce.png)
3. 立即啟動 Docker 服務

    ```bash
    systemctl start docker
    ```

    ![2startdocker](https://user-images.githubusercontent.com/3851540/40274844-1b5180c2-5c14-11e8-92fe-357cfc9be0b1.png)
4. 確認安裝成功

    ```bash
    docker version
    ```

    ![3dockerversion](https://user-images.githubusercontent.com/3851540/40274845-1b7ebf7e-5c14-11e8-84be-5ec291f65ba6.png)

## 安裝 kubeadm, kubelet 跟 kubectl

> 記得執行指令前加上 `sudo` 或是切換為 `root`

1. 將 yum 來源位置覆寫至 kubernetes.repo

    > 這邊要留意一下從 Windows 貼上因為換行符號與 linux 不同，可能會有問題

    ```bash
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF
    ```

    ![4catrepo](https://user-images.githubusercontent.com/3851540/40274846-1ba90b62-5c14-11e8-98fc-3ca841559edd.png)
2. 關閉 `SELinux`

    > `SELinux` 是 `Security Enhanced Linux` 縮寫，原意是為了加強安全性避免被攻擊，但因為 container 需要存取 host 上的檔案系統，而目前 `kubelet` 暫時未支援 `SELinux`，需先行關閉

    ```bash
    setenforce 0
    ```

3. 安裝 `kubelet`，`kubeadm`，`kubectl`
    - kubelet

        > 安裝於 cluster 所有機器上的元件，用來啟動 pod 跟 container
    - kubeadm

        >是 Kubernetes 官方提供的快速安裝和初始化 Kubernetes cluster 的工具
    - kubectl

        > Kubernetes 的命令列工具

    ```bash
    yum install -y kubelet kubeadm kubectl
    ```

4. 設定 `kubelet` 在系統開機時自動啟動並立即啟動 `kubelet` 服務

    ```bash
    systemctl enable kubelet && systemctl start kubelet
    ```

    ![5startkubelet](https://user-images.githubusercontent.com/3851540/40274847-1bcecc58-5c14-11e8-8b87-59095db6770c.png)
5. 確認安裝成功

    ```bash
    kubeadm version
    ```

    ![6kubeadmversion](https://user-images.githubusercontent.com/3851540/40274848-1bf593ba-5c14-11e8-9b58-4c3f81323d97.png)

## 關閉系統 Swap

>Swap 類似 Windows 中的虛擬記憶體，從 Kubernetes 1.8 開始要求關閉系統的 Swap

- 關閉 Swap

    ```bash
    swapoff -a
    ```

- 修改 `/etc/fstab`，避免 `Swap` 自動掛載

    ```bash
    sed -e '/swap/ s/^#*/#/' -i /etc/fstab
    ```

- 確認關閉

    ```bash
    free -m
    ```

    ![7swapoff](https://user-images.githubusercontent.com/3851540/40274849-1c1c6134-5c14-11e8-8ff6-7a04f99a88ee.png)

## 建立 cluster

1. 初始化 master

    > `--pod-network-cidr` 是為後面的 Pod 網路套件 (`flannel`) 做準備

    ```bash
    kubeadm init --pod-network-cidr=10.244.0.0/16
    ```

    ![8kubeadminit](https://user-images.githubusercontent.com/3851540/40274850-1c452dda-5c14-11e8-8b31-179294019c5b.png)
2. 確保有權限執行 `kubectl`
    - 非 root user

        ```bash
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```

    - root user

        ```bash
        export KUBECONFIG=/etc/kubernetes/admin.conf
        ```

3. 安裝 Pod 網路套件

    > 這邊以 `flannel` 為例

    ```bash
    kubectl apply --namespace kube-system -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ```

    ![9flannel](https://user-images.githubusercontent.com/3851540/40274851-1c6af75e-5c14-11e8-85ab-37172fc33146.png)

## 加入其他 Node

1. 新的 Node 上需先完成前面的 `安裝 Docker`，`安裝 Kubernetes`，`關閉系統 Swap`
2. 執行 join 指令

    > 指令內容會在 master 執行 `kubeadm init` 時產生

    ```bash
    kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
    ```

    ![10joinscript](https://user-images.githubusercontent.com/3851540/40274947-6ce16cd4-5c16-11e8-802e-ce87742ab8a7.png)

    - 出現如下錯誤訊息

        ```log
        Failed to request cluster info, will try again: [Get https://x.x.x.x:xxxx/api/v1/namespaces/kube-public/configmaps/cluster-info: dial tcp x.x.x.x:xxxx: getsockopt: no route to host
        ```

        ![10joinerror](https://user-images.githubusercontent.com/3851540/40274852-1c9c7d56-5c14-11e8-9c37-fb4c0916050d.png)
    - 請先將 master 的防火牆關閉

        ```bash
        systemctl disable firewalld
        systemctl stop firewalld
        ```

        ![11firewall](https://user-images.githubusercontent.com/3851540/40274853-1cc6c3c2-5c14-11e8-8abb-3828046a0160.png)
3. 加入成功

    ![12nodejoined](https://user-images.githubusercontent.com/3851540/40274854-1cf07118-5c14-11e8-8e5f-bab2d755965f.png)
4. 至 master 上確認

    ```bash
    kubectl get nodes
    ```

    ![13getnodes](https://user-images.githubusercontent.com/3851540/40274855-1d1d285c-5c14-11e8-9ccf-0d5171327026.png)

## 心得

大致安裝流程與 [Windows 慣用者如何在 Ubuntu Server 16.04 LTS 上使用 kubeadm 架設 Kubernetes (K8s)](/ubuntu-kubeadm-kubernetes) 相同，只是語法上跟 Ubuntu 有些不同，另外需要特別留意的是 CentOS 安全性較高，需要自行關閉

但對於關閉 `SELinux` 與防火牆，我還是覺得有點害怕，或許日後的版本會再做調整吧

## 參考資訊

1. [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
2. [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
3. [linux yum 命令](http://www.runoob.com/linux/linux-yum.html)
4. [systemd (正體中文)](https://wiki.archlinux.org/index.php/Systemd_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87))
5. [kubeadm join - Failed to request cluster info - getsockopt: no route to host](https://groups.google.com/forum/#!topic/kubernetes-dev/epes4exK6Ms)
6. [sysctl 讀取 / 修改 Kernel 變數](https://www.phpini.com/linux/sysctl-read-modify-kernel-var)
7. [Windows 慣用者如何在 Ubuntu Server 16.04 LTS 上使用 kubeadm 架設 Kubernetes (K8s)](/ubuntu-kubeadm-kubernetes)
