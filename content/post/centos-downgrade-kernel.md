---
title: "CentOS 8 降級 Linux Kernel 5"
date: 2020-10-22T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["Linux"]
slug: "centos-downgrade-kernel"
---

## CentOS 8 降級 Linux Kernel 5

之前筆記 [CentOS 7 升級 CentOS 8 連帶安裝 Linux Kernel 5](/centos7-upgrade-centos8-linux-kernel5/) 興高采烈地把環境升級到最新版本：CentOS 8 + Linux Kernel 5.9.1，結果遇到 Ansible 的 service module 出現 `Service is in unknown state` 的 issue，據 GitHub 上的討論串 [Service is in unknown state](https://github.com/ansible/ansible/issues/71528) 看來並沒有真的解決，所以決定先降版，快速紀錄一下過程與語法

## 基本環境說明

1. macOS Catalina 10.15.7
2. VrtualBox 6.1.12

    - bridge 網路

    - 開啟對外

        ```bash
        nmcli connection modify enp0s3 \
        connection.autoconnect yes \
        ipv4.method auto
        ```

3. 原環境

    - CentOS 8.2.2004
    - Linux Kernel 5.9.1

        ![1old](https://user-images.githubusercontent.com/3851540/96956338-6e673780-152a-11eb-8013-a756d3fd357d.png)

4. 新環境
    - CentOS 8.2.2004
    - Linux Kernel 5.7.12

## 操作步驟

1. 移除 `kernel-ml`

    ```bash
    rpm -e `rpm -q kernel-ml`
    ```

2. 安裝目標版本的 kernel

    > 這個步驟我卡最久的是找到適合的 kernel rpm，我個人是在 [Index of /elrepo/archive/kernel/el8/x86_64/RPMS/](http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/) 不確定有沒有問題，大家自行斟酌

    - 安裝 kernel-core

        ```cs
        dnf install -y http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/kernel-ml-core-5.7.12-1.el8.elrepo.x86_64.rpm
        ```

    - kernel-modules

        ```cs
        dnf install -y http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/kernel-ml-modules-5.7.12-1.el8.elrepo.x86_64.rpm
        ```

    - kernel

        ```cs
        dnf install -y http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/kernel-ml-5.7.12-1.el8.elrepo.x86_64.rpm
        ```

3. 重新啟動

    ```cs
    reboot
    ```

    ![2new](https://user-images.githubusercontent.com/3851540/96956340-6effce00-152a-11eb-940c-4cd8b33fb7af.png)

## 如果需要刪除舊版 kernel

這是個人測試用語法，留個紀錄

```bash
rpm -e `rpm -q kernel-ml-5.7.12-1.el8.elrepo.x86_64 kernel-ml-modules-5.7.12-1.el8.elrepo.x86_64kernel-ml-core-5.7.12-1.el8.elrepo.x86_64`
```

## 心得

語法不多，需要留意的是執行安裝順序，之前測試時有遇到順序問題造成的錯誤，另外是 rpm 來源是否安全也是需要特別注意的，但我個人目前還不會自己 build linux kernel，只好先相信它了

## 參考資訊

1. [CentOS 7 升級 CentOS 8 連帶安裝 Linux Kernel 5](/centos7-upgrade-centos8-linux-kernel5/)
2. [Service is in unknown state](https://github.com/ansible/ansible/issues/71528)
3. [Index of /elrepo/archive/kernel/el8/x86_64/RPMS/](http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/)
