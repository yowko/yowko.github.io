---
title: "CentOS 7 安裝 Linux Kernel 5"
date: 2020-10-30T21:30:00+08:00
lastmod: 2020-10-30T21:30:31+08:00
draft: false
tags: ["Linux"]
slug: "centos7-install-linux-kernel5"
---

## CentOS 7 安裝 Linux Kernel 5

為了 server 管理方便傾向不升級 CentOS 8 與 Linux kernel 5，但在效能議題下只好犧牲一點管理上的方便性，於是決定暫時不升級 CentOS 8，而是在 CentOS 7 下使用 Linux kernel 5，快速紀錄一下備用

筆記中有紀錄到升級 mainline kernel 的方法，以及使用特定 kernel 版本 rpm 的方式，可以依實際需求選擇

## 基本環境說明

1. macOS Catalina 10.15.6
2. VrtualBox 6.1.12

    - bridge 網路

    - 開啟對外

        ```bash
        nmcli connection modify enp0s3 \
        connection.autoconnect yes \
        ipv4.method auto
        ```

3. CentOS 7.8.2003
4. 原環境
    - Linux Kernel 3.10.0

        ![1oskernel](https://user-images.githubusercontent.com/3851540/95656343-67aaee80-0b40-11eb-9f55-df9819e957ca.png)

5. 新環境
    - Linux Kernel 5.7.14

## 安裝 mainline kernel (最新版)安裝步驟

1. 移除舊 kernel

    ```bash
    rpm -e `rpm -q kernel`
    ```

2. 安裝 elrepo rpm

    ```bash
    yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
    ```

3. 安裝 kernel 與 kernel-header

    ```bash
    yum --enablerepo=elrepo-kernel -y install kernel-ml kernel-ml-headers
    ```

4. 將開機選單倒數秒數設為 `0`  (optional)

    ```bash
    sed -i 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0/g' /etc/default/grub
    ```

5. 設定預設開機選項

    ```bash
    sed -i 's/GRUB_DEFAULT=saved/GRUB_DEFAULT=0/g' /etc/default/grub
    ```

6. 重新產生開機設定

    ```bash
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

7. 重新開機

    ```bash
    reboot
    ```

8. 安裝結果

    ```bash
    cat /etc/redhat-release && uname -r
    ```

    ![1kernelml](https://user-images.githubusercontent.com/3851540/97735402-40c75300-1b15-11eb-8e14-1bb356719ef2.png)

## 安裝非 mainline kernel 安裝步驟

1. 移除舊 kernel

    ```bash
    rpm -e `rpm -q kernel`
    ```

2. 安裝 kernel 5.7.12

    > 這個步驟我卡最久的是找到適合的 kernel rpm，我個人是在 [Index of /elrepo/archive/kernel/el8/x86_64/RPMS/](http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/) 不確定有沒有問題，大家自行斟酌

    ```bash
    yum install -y http://repos.ord.lax-noc.com/elrepo/archive/kernel/el7/x86_64/RPMS/kernel-ml-5.7.12-1.el7.elrepo.x86_64.rpm
    ```

3. 安裝 kernel-header

    ```bash
    yum install -y http://repos.ord.lax-noc.com/elrepo/archive/kernel/el7/x86_64/RPMS/kernel-ml-headers-5.7.12-1.el7.elrepo.x86_64.rpm
    ```

4. 將開機選單倒數秒數設為 `0` (optional)

    ```bash
    sed -i 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0/g' /etc/default/grub
    ```

5. 設定預設開機選項

    ```bash
    sed -i 's/GRUB_DEFAULT=saved/GRUB_DEFAULT=0/g' /etc/default/grub
    ```

6. 重新產生開機設定

    ```bash
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

7. 重新開機

    ```bash
    reboot
    ```

8. 安裝結果

    ```bash
    cat /etc/redhat-release && uname -r
    ```

    ![2kernel](https://user-images.githubusercontent.com/3851540/97735410-42911680-1b15-11eb-9b2b-930f100c03bf.png)

## 心得

因為 Kubernetes 搭配的 Calico 網路套件，有個優化的設定需要 kernel 4.20 以上，所以決定將 server 的 kernel 都升至 5，原本是想使用最新版的 kernel (2020/10/30 時是 kernel 5.9)，但 kernel 5.8 以上會讓 ansible 的 service module 異常，所以暫時就改安裝 kernel 5.7.12，這就是今天這份筆記的由來

## 參考資訊

1. [Linux：CentOS 7 更新Kernel 5.x及開啟BBR](https://blog.cwlove.idv.tw/linux-centos-update-kernel-4-19-open-bbr/)
2. [How To Install Linux Kernel 5.x on CentOS 7](https://computingforgeeks.com/install-linux-kernel-5-on-centos-7/)
