---
title: "透過 Kubespray 來架設 Kubernetes"
date: 2019-07-13T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "kubespray-kubernetes"
---

## 透過 Kubespray 來架設 Kubernetes

最近剛好有機會可以參與專案的部署作業，雖然過去也持續進行過不同方式的 CI/CD，但真正部署至 Kubernetes 上則是全新的體驗，為了降低扯團隊後腿的機會，免不了要自己練習一下先，在細節不少加上本來掌握度就不高下狀況可謂是相當慘烈呀，於是一搞定就馬上筆記，加深印象

過去筆記 [(CentOS ) 上使用kubeadm 架設Kubernetes (K8s)](/centos-kubeadm-kubernetes/) 提到的 kubeadm 與 Kubespray 都是 Google open source 的工具，其中 Kubespray 主要核心是透過 Ansible 來執行相關操作，雖說與 Kubeadm 都有細微操作被包裝的問題，但相較之下 Kubespray 因為搭配了 Ansible，讓靈活度與門檻降低了些

## 基本環境說明

> 以下 VM 透過 Azure 建立

1. CentOS 7.6 * 3

    - Standard B2ms
    - 2 vcpus, 8 GiB memory
    - 200 GB Standard SSD

2. VM 配置說明

    > 請確保各個 node 間可以透過 host name 連線

    Name|IP|用途
    :---|:---|:---
    node1| 10.0.0.4|ansible-client,master,etcd
    node2| 10.0.0.5|node,etcd
    node3| 10.0.0.6|node,etcd

3. Kubespray 2.10
4. Kubernetes 1.14.3

## VM 基礎環境設定

1. 關閉 selinux

    > 執行對象：所有 VM

    ```bash
    setenforce 0
    sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    ```

    - `setenforce 0` 暫時關閉
    - `sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux` 永久關閉，需重新開機

2. 關閉系統 Swap 交換空間

    > 執行對象：所有 vm

    ```bash
    swapoff -a && echo "vm.swappiness=0" >> /etc/sysctl.conf && sysctl -p && free –h
    ```

3. 閉閉防火牆

    > 執行對象：所有 vm

    ```bash
    systemctl disable firewalld && systemctl stop firewalld
    ```

4. 啟用 iptable filter 的 FOWARD

    > 執行對象：所有 vm

    ```bash
    iptables -P FORWARD ACCEPT
    ```

5. 啟用 bridge-nf-call-iptablesx

    > 這是同事提醒的設定，不啟用的話某些網路功能會受限 (我還沒搞清問題，先紀錄起來)

    ```bash
    modprobe br_netfilter
    echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
    ```

6. 設定免密碼登入

    > 執行對象：ansible-client ;在 ansible-client 產生 key，並複製至各個 vm，避免執行 ansible 部署時出問題

    ```bash
    ssh-keygen -t rsa -N ""

    ssh-copy-id -i /root/.ssh/id_rsa.pub 10.0.0.4
    ssh-copy-id -i /root/.ssh/id_rsa.pub 10.0.0.5
    ssh-copy-id -i /root/.ssh/id_rsa.pub 10.0.0.6  
    ```

7. 安裝基本軟體

    > 執行對象：ansible-client

    ```bash
    yum -y install epel-release && yum -y install https://centos7.iuscommunity.org/ius-release.rpm && yum clean all && yum makecache && yum install -y python-pip python36 python-netaddr python36-pip ansible git
    ```

## 透過 Kubespray 安裝

> 執行對象： ansible-client

1. 下載 Kubespray

    > 2019/7/14 測試： `master` 仍無法順利完成安裝，需使用 `release-2.10` branch 最高僅能安裝 `Kubernetes v1.14.3`

    > 2019/9/6 測試：使用 `release-2.11` 可以順利安裝 `Kubernetes v1.15.3`

    ```bash
    git clone https://github.com/kubernetes-sigs/kubespray.git -b release-2.11
    ```

    > 2019/12/5 測試：使用 `release-2.12` 可以順利安裝 `Kubernetes v1.16.3`

    ```bash
    git clone https://github.com/kubernetes-sigs/kubespray.git -b release-2.11
    ```

2. 安裝相依套件

    ```bash
    cd kubespray

    pip install -r requirements.txt
    ```

3. 調整 Kubespray 參數

    - 複製一份 config

        > `{k8s}` 名稱可自訂，以下以 `k8s` 為例示範

        ```bash
        cp -rfp inventory/sample inventory/k8s
        ```

    - 修改 `inventory/k8s/inventory.ini`

        ```yml
        [all]
        node1 ansible_host=10.0.0.4
        node2 ansible_host=10.0.0.5
        node3 ansible_host=10.0.0.6

        [kube-master]
        node1

        [etcd]
        node1
        node2
        node3

        [kube-node]
        node1
        node2
        node3

        [k8s-cluster:children]
        kube-master
        kube-node
        ```

    - 修改 `inventory/k8s/group_vars/all/all.yml`

        ```yml
        loadbalancer_apiserver_localhost: true

        kubelet_load_modules: true
        ```

4. 執行 Kubespray 安裝 Kubernetes

    ```bash
    ansible-playbook -i inventory/k8s/inventory.ini cluster.yml -b -v -k
    ```

> Kubernetes 的版本紀錄在 `inventory/k8s/group_vars/k8s-cluster/k8s-cluster.yml`

## 心得

看了幾篇 kubespray 安裝的網路文章，大致上理解操作流程與步驟，不過細節幾乎不同：像是 `更新 cpu kernel`、`inventory.ini` 值的設定...etc，對於一個新入門的人來說，我自己覺得門檻不低，另外一個讓我決定要自己紀錄一篇的重大理由是：看的大部份文章都需要 `科學上網`，或是調整 docker registry 位置，這對我來說是種雜訊

我照著文章操作過不下十次，VM 建了又刪、刪了又建，一直無法成功，老是卡在 etcd 的 health check，每次一跑都要個十幾二十分鐘，最後靠著  Kubernetes 大神同事的幫忙，終於發現原來是 kubespray master 的 issue 造成的，改用 release branch 就沒問題，感謝同事的協助

就個人使用體驗來說，kubespray 確實比 kubeadm 好些，多數設定都可以透過 yaml 檔案來調整，指令使用也相對較簡潔，不過還是有不少動作被包在 ansible 中，從這點來看 ansible 至少還是比較通用些

## 參考資訊

1. [kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
2. [CentOS 關閉 selinux](https://blog.xuite.net/tolarku/blog/195633562-CentOS+%E9%97%9C%E9%96%89+selinux)
3. [Kubernetes 1.13.1](https://jicki.me/kubernetes/docker/2018/12/21/k8s-1.13.1-kubespray/)
4. [使用Kubespray 2.8.3部署生產可用的Kubernetes集群（1.12.5）](http://www.itmuch.com/install/kubernetes-deploy-by-kubespray2.8.3/)
5. [使用Kubespray自動化部署Kubernetes 1.13.1](https://www.kubernetes.org.cn/5012.html)
6. [centos7上安裝docker以及一些小問題](https://www.itread01.com/p/1385936.html)
