---
title: "CentOS 7 升級 CentOS 8 連帶安裝 Linux Kernel 5"
date: 2020-10-10T21:30:00+08:00
lastmod: 2020-10-10T21:30:31+08:00
draft: false
tags: ["Linux"]
slug: "centos7-upgrade-centos8-linux-kernel5"
---

## CentOS 7 升級 CentOS 8 連帶安裝 Linux Kernel 5

最近在 review 團隊各項技術的版本，準備再來次大升級，其中的較大的內容就是基礎環境的部份，原本之前就想升級 Kubernetes 1.18 但無奈 Kubespray 當時尚未支援，過了幾個月再次 review 時已確認可以正常使用，只是這次多 review 了底層 kernel 與 OS 的版本，經過簡單討論後決定將 CentOS7 升級至 CentOS8，連帶一併將原本使用的 Linux Kernel 4 升成 Linux Kernel 5

過去工作經驗中，遇到需要這麼大程度的升級，都是直接請 server team 同事先提供新機器並預先安裝新的 OS，我們把程式部署上去驗證沒問題後再挑個黃道吉日將流量跟設定調整過去新機器上，舊機器就放個幾周沒問題就還個 server team，可是這次因為沒有多餘的機器可以做這樣的操作，得要直接進行升級，為了加速之後升級的速度，我來筆記一下

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

3. 原環境

    - CentOS 7.8.2003
    - Linux Kernel 3.10.0

        ![1oskernel](https://user-images.githubusercontent.com/3851540/95656343-67aaee80-0b40-11eb-9f55-df9819e957ca.png)

4. 新環境
    - CentOS 8.2.2004 
    - Linux Kernel 5.8.14

## 升級步驟

1. 安裝 Epel yum Repository

    ```bash
    yum install -y epel-release
    ```

    ![2installepel](https://user-images.githubusercontent.com/3851540/95656344-6974b200-0b40-11eb-962f-b684b8795198.png)

2. 安裝升級過渡用套件

    ```bash
    yum install -y yum-utils rpmconf
    ```

    ![3yumrpmconf](https://user-images.githubusercontent.com/3851540/95656346-6aa5df00-0b40-11eb-8dff-0405c5a7da5f.png)

3. 清除已不再被依賴的套件

    ```bash
    package-cleanup --leaves|tail -n +2|xargs yum remove -y
    ```

    ![5removeleaves](https://user-images.githubusercontent.com/3851540/95656348-6b3e7580-0b40-11eb-9d69-e914cd6deadf.png)

4. 安裝 dnf

    ```bash
    yum install -y dnf
    ```

    ![6installdnf](https://user-images.githubusercontent.com/3851540/95656349-6bd70c00-0b40-11eb-8877-3dd853299f08.png)

5. 移除 yum

    ```bash
    dnf remove -y yum yum-metadata-parser
    ```

    ![7removyum](https://user-images.githubusercontent.com/3851540/95656350-6bd70c00-0b40-11eb-9986-a1bfa18c63f6.png)

6. 使用 dnf 更新 centos 7 系統

    ```bash
    dnf upgrade -y
    ```

    ![8dnfupgrade](https://user-images.githubusercontent.com/3851540/95656351-6c6fa280-0b40-11eb-8436-35d1d52016f4.png)

7. 升級 CentOS 8 RPM

    ```bash
    dnf install -y http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-repos-8.2-2.2004.0.1.el8.x86_64.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-release-8.2-2.2004.0.1.el8.x86_64.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-gpg-keys-8.2-2.2004.0.1.el8.noarch.rpm
    ```

    ![9centos8](https://user-images.githubusercontent.com/3851540/95656352-6d083900-0b40-11eb-8298-8effaa245d19.png)

8. 匯入 gpg key

    ```bash
    rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    ```

9. 升級 epel

    ```bash
    dnf upgrade -y epel-release
    ```

    ![10upgradeepel](https://user-images.githubusercontent.com/3851540/95656353-6da0cf80-0b40-11eb-94da-0e43fa104a46.png)

10. 清除舊 kernel

    ```bash
    rpm -e `rpm -q kernel`
    ```

11. 移除衝突套件

    ```bash
    rpm -e --nodeps sysvinit-tools
    ```

12. 升級 CentOS 8

    > 移除 python 3 的動作是我在數十次安裝中都會遇到的問題，所以先加上來，如果有疑慮可以先執行升級，待出現衝突再移除即可

    - 移除 python 3 

        ```bash
        dnf remove -y python3
        ```

        ![11removepython3](https://user-images.githubusercontent.com/3851540/95656354-6da0cf80-0b40-11eb-9677-037f2f081dbe.png)

    - 升級 CentOS 8

        ```bash
        dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
        ```

        ![12upgradecentos](https://user-images.githubusercontent.com/3851540/95656355-6e396600-0b40-11eb-9837-06f4a98674bd.png)

13. 升級 kernal 至 mainline

    - 加入  rpm

        ```bash
        dnf install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
        ```

    - 安裝 mainline lernel

        ```bash
        dnf --enablerepo=elrepo-kernel -y install kernel-ml
        ```

        ![13kernelinstalled](https://user-images.githubusercontent.com/3851540/95656356-6ed1fc80-0b40-11eb-80a6-7993052cffbc.png)

14. 設定開機選單

    - 讓開機時間不要倒數

        ```bash
        sed -i 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0/g' /etc/default/grub
        ```

    - 重新產生開機設定

        ```bash
        grub2-mkconfig -o /boot/grub2/grub.cfg
        ```

15. 重新開機套用新的 kernel

    ```bash
    reboot
    ```

16. 實際效果

    ```bash
    cat /etc/redhat-release &&  uname -r
    ```

    ![14result](https://user-images.githubusercontent.com/3851540/95656357-6f6a9300-0b40-11eb-9480-3a3184b2ae5d.png)

## 心得

一開始我是透過 Azure VM 來進行安裝，但持續遇到開單選單無法成功建立的錯誤，後來才想到會不會是 Azure VM 的架構不允許我做這樣的升級行為，因為雲端 VM 的建立也是基於 image 的基礎上，後來使用 VirtualBox 來建立測試環境，的確也順利解決問題

以下是我測試過程中遇到的幾個問題，我簡單紀錄一下問題與解決方式

1. RPM 衝突

    > 這個我是在安裝 centos8 的 rpm 後遇到的

    - 錯誤截圖

        ![15rpmconf](https://user-images.githubusercontent.com/3851540/95656359-70032980-0b40-11eb-98b6-fbb5cb8304ac.png)

    - 解決方式

        > 透過 `重新設定 rpm config` 使用新的 rpm (互動選項我選 `Y` - 複寫舊設定)

        ```bash
        rpmconf -a
        ```

2. Kernel 執行中

    > 我不確定是不是只有在 Azure VM 才會遇到，或者就是這個操作造成後續開機選單異常

    - 錯誤截圖
        ![16rmkernelerror](https://user-images.githubusercontent.com/3851540/95656360-709bc000-0b40-11eb-93e3-bf219de4817c.png)
    - 解決方式

        > 該不該這麼做，請自行斟酌

        ```bash
        dnf -y remove hypervvssd hypervkvpd kmod-kvdo && rpm -e hypervfcopyd
        ```

## 參考資訊

1. [How to upgrade CentOS 7 to CentOS 8 Server](https://www.centlinux.com/2020/01/how-to-upgrade-centos-7-to-8-server.html)
2. [How to Upgrade Centos 7 to 8](https://www.howtoforge.com/how-to-upgrade-centos-7-core-to-8/)
3. [[CentOS 7] 核心升級免苦惱，三行指令就搞定](http://blog.itist.tw/2016/03/how-to-upgrade-newest-kernel-on-centos-7.html)
4. [How to Change the default kernel (boot from old kernel) in CentOS/RHEL 8](https://www.thegeekdiary.com/how-to-change-the-default-kernel-boot-from-old-kernel-in-centos-rhel-8/)
5. [調整Linux GRUB開機選單的倒數秒數](http://slashview.com/archive2019/20190524.html)
6. [VirtualBox 中 centos7 下 ping 命令出現 Network is unreachable 問題的解決方法](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/481447/)
