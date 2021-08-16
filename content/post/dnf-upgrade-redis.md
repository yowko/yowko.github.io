---
title: "使用 dnf 升級 redis 版本"
date: 2021-08-15T21:30:00+08:00
lastmod: 2021-08-15T21:30:00+08:00
draft: false
tags: ["CentOS","redis"]
slug: "dnf-upgrade-redis"
---

## 使用 dnf 升級 redis 版本

又到了定期 review 各環境中使用服務與軟體版本的時間，這次發現 production 的 redis 從安裝時的 6.0.14 至今皆未升級，內網環境因為之前調整機器有重新安裝過，已是 6.2.4 的版本，所以排定計劃要將 production redis 升級至 6.2.4 (最新版本為 6.2.5，但因未在內網做過功能驗證，故只升級至 6.2.4)

因為當時安裝 redis 用了 dnf module，讓更新的動作有些不同，為了節省日後要重新回憶的時間，快速筆記一下

## 基本環境說明

1. Azure VM : 標準 B2s (2 個 vcpu，4 GiB 記憶體)
2. OS image : CentOS 8.2 - Gen1
3. redis

    - 6.0.14
    - 6.2.24

    > 相關安裝方式可以參考之前筆記 [使用 dnf 安裝 Redis 6](/dnf-install-redis6)

    ```bash
    dnf module enable -y redis:remi-6.0 && \
    dnf install -y --enablerepo=remi redis-0:6.0.14-1.el8.remi.x86_64
    ```

## 升級步驟

1. 最出所有 module stream

    ```bash
    dnf module list redis
    ```

    ![1modulelist](https://user-images.githubusercontent.com/3851540/129505292-bd5b0132-9943-41af-abe5-c9fc6a1f0eb6.png)

2. 確認需要的版本在哪一個 stream

    ```bash
    dnf module info redis:remi-*
    ```

    ![2moduleinfo](https://user-images.githubusercontent.com/3851540/129505298-069050b9-bdbe-4035-83ad-9ddd2d8664e4.png)

3. 重設 module

    ```bash
    dnf module reset -y redis
    ```

    - 未重設 module 直接使用其他 stream 的錯誤訊息

        ```txt
        Last metadata expiration check: 0:13:56 ago on Mon 16 Aug 2021 02:23:18 AM UTC.
        Dependencies resolved.
        The operation would result in switching of module 'redis' stream 'remi-6.0' to stream 'remi-6.2'
        Error: It is not possible to switch enabled streams of a module.
        It is recommended to remove all installed content from the module, and reset the module using 'dnf module reset <module_name>' command. After you reset the module, you can install the other stream.
        ```

    - 未重設 module 直接使用其他 stream 的錯誤截圖

        ![3enableerror](https://user-images.githubusercontent.com/3851540/129505302-a0572494-e909-4d77-b610-8494f36a63bc.png)

4. 重新啟用指定的 module stream

    ```bash
    dnf module enable -y redis:remi-6.2
    ```

5. 使用指定的 module stream 安裝指定本

    > 指定版本的安裝方式，可以參考之前筆記 [使用 dnf 透過 module 安裝指定版本套件](/dnf-module-install-specific-artifact/)

    ```bash
    dnf install -y --enablerepo=remi redis-0:6.2.4-1.el8.remi.x86_64
    ```

6. 實際效果

    ```bash
    redis-cli --version && redis-server --version
    ```

    ![4result](https://user-images.githubusercontent.com/3851540/129505305-52253209-710c-4a16-a652-7539ebe77ecb.png)

## 心得

原本以為升級非常簡單，就是把 [避免執行 dnf update 時連帶升級特定套件](dnf-update-exclude/) 解除，然後像以前 yum 一樣：執行 `yum update redis`  結果就如同筆記中提到的，因為用了 dnf module，而需要的版本又位於不同 stream 中，無法直接升級成指定版本，幸虧之前有花時間好好看了 dnf module 的使用方式，沒有卡住，但實在沒有信心下次遇到類似狀況還能快速搞定，還是先筆記一下準備拯救自己比較實在

## 參考資訊

1. [使用 dnf 安裝 Redis 6](/dnf-install-redis6)
2. [使用 dnf 透過 module 安裝指定版本套件](/dnf-module-install-specific-artifact/)
3. [避免執行 dnf update 時連帶升級特定套件](dnf-update-exclude/)
