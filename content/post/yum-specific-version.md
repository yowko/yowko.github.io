---
title: "yum 安裝指定版本套件"
date: 2020-08-16T16:30:00+08:00
lastmod: 2020-08-16T21:30:31+08:00
draft: false
tags: ["Linux"]
slug: "yum-specific-version"
---

## yum 安裝指定版本套件

這個問題起因於要追查線上問題，但另建環境卻怎樣也模擬不出錯誤，後來才發現另建的環境使用的套件是較新版本，原本的 bug 也被修復；另個常見狀況是因為新版本的功能尚未穩定，或是有 breaking change 需要同步調整程式碼，所以必需鎖定在某個特定版本上

雖說是個簡單語法，但使用頻率不高，讓我每次都要重新查語法，所以趁著休假空檔簡單紀錄一下

## 基本環境說明

1. Azure VM B2s (2 vcpu,4GiB memory)
2. OpenLogic 7.7
3. Redis 5.0.9

## 安裝步驟

1. 確認可用版本

    ```bash
    yum list redis --showduplicates
    ```

    ![1listduplicate](https://user-images.githubusercontent.com/3851540/90332015-d447b800-dfeb-11ea-952d-bb1e748d52ff.png)

2. 版本資訊拆解

    第一個步驟取得的資訊內容如下

    > `{package name}-{arch}    {version}-{release}    {repo name}`

    ![2detailinfo](https://user-images.githubusercontent.com/3851540/90332017-d6aa1200-dfeb-11ea-87af-e7eb1605a058.png)

3. 確認詳細資訊 (optional)

    > 這個步驟可省略，僅用來確認 package 的各項詳細資訊

    - 完整語法：

        ```bash
        yum info {package name}-{version}-{release}.{arch}
        ```

    - 完整語法範例

        ```bash
        yum info redis-5.0.9-1.el7.remi.x86_64
        ```

        ![3infofull](https://user-images.githubusercontent.com/3851540/90332018-d742a880-dfeb-11ea-925b-efbb8dfd739d.png)

    - 簡易語法：

        ```bash
        yum info {package name}-{version}
        ```

    - 簡易語法範例：

        ```bash
        yum info redis-5.0.9
        ```

        ![4infosimple](https://user-images.githubusercontent.com/3851540/90332019-d7db3f00-dfeb-11ea-8373-b111fd8917b5.png)

4. 實際安裝

    - 完整語法：

        ```bash
        yum install -y {package name}-{version}-{release}.{arch}
        ```

    - 完整語法範例

        ```bash
        yum install -y redis-5.0.9-1.el7.remi.x86_64
        ```

        ![5installfull](https://user-images.githubusercontent.com/3851540/90332020-d7db3f00-dfeb-11ea-95de-966be6b3b198.png)

    - 簡易語法：

        ```bash
        yum install -y {package name}-{version}
        ```

    - 簡易語法範例：

        ```bash
        yum install -y redis-5.0.9
        ```

        ![5installsimple](https://user-images.githubusercontent.com/3851540/90332021-d873d580-dfeb-11ea-8d0d-061c14df2d10.png)

## 心得

透過 yum 安裝指定版本套件，有時候還是無法安裝到想要的版本；舉例來說：幾個月前當時我們用的最新版是 `redis 5.0.7` 但現在最新版變成了 `redis 6.0.6`，這期間短暫出現過的 `redis 6.0.4` ``redis 6.0.3` 甚至於 `redis 5.0.7` 都已經沒出現 yum list 中了，redis 5 系統僅留下 `redis 5.0.9`

以結果來看，推測是只留下大版本中的最後一版，所以如果真的有不能升級的需求，可能還是自行保留 rpm 最為妥當

## 參考資訊

1. [CentOS / RHEL : How to install a specific version of rpm package using YUM](https://www.thegeekdiary.com/centos-rhel-how-to-install-a-specific-version-of-rpm-package-using-yum/)
