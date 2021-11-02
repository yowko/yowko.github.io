---
title: "安裝 Redis 時遇到的錯誤排除"
date: 2020-02-09T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Linux","Redis","Debug"]
slug: "install-redis-error"
---

## 安裝 Redis 時遇到的錯誤排除

之前筆記 [使用 Yum 安裝 Redis 5](/yum-install-redis5) 提到最近嘗試撰寫 script 在 CentOS 上建立 Redis 環境，也才發現幾個 yum package 的版本差異與 epel、ius 兩個 yum repository 的相依關係，除此之外過程中還遇到之前人工安裝時沒遇到的問題，覺得值得紀錄一下

## 基本環境說明

1. Azure VM B1s (1 vcpu,1GiB memory)
2. CentOS-based 7.7
3. Redis 5.0.7
4. epel-release.noarch 0:7-11
5. ius-release.noarch 0:2-1.el7.ius

## 錯誤訊息與解決方式

1. 錯誤一

    - 現象：
  
        >slave 或是某個 redis instnace service 無法啟動，個人經驗較常出現在 slave 或是第二個啟動的 redis instance 上

    - 錯誤訊息內容

        ```txt
        Could not create server TCP listening socket 127.0.0.1:6380: bind: Permission denied
        ```

    - 錯誤截圖

        ![1tcpsocketdeny](https://user-images.githubusercontent.com/3851540/74103978-7a24f080-4b8b-11ea-9346-1942dfec44ed.png)

    - 解決方式

        > 關閉 `SELinux`，但有安全疑慮時則建議使用 `semanage` 調整，以下僅示範關閉 `SELinux` 用法
        >
        > - `setenforce 0` 是針對當前 server 狀態
        > - `/etc/selinux/config` 是避免重開機時 server 狀態被重置

        ```bash
        setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
        ```

2. 錯誤二

    - 現象：
  
        >master、slave 服務正常啟動，但 master 執行 `redis-cli info replication` 顯示無連線的 slave;而 slave 執行 `redis-cli info replication` 則顯示 master 的狀態一直是 `down`，使用 `journalctl -xe` 檢查 master log 會出現錯誤訊息

    - 錯誤訊息內容

        ```txt
        Failed opening the RDB file dump.rdb (in server root dir /) for saving: Permission denied

        Background saving error

        SYNC failed. BGSAVE child returned an error
        ```

    - 錯誤截圖

        ![2datapermission](https://user-images.githubusercontent.com/3851540/74103981-7db87780-4b8b-11ea-9a20-c57921887d72.png)

    - 解決方式

        >將 redis config 中的 `data` (預設為 `/`) 指向執行 redis service 使用者 (使用 yum 安裝，預設 user 為 `redis`) 有權限的路徑，權限需為 `755`

3. 錯誤三

    - 現象：
  
        >master、slave 服務正常啟動， master 與 slave 間的 sync 狀態也正常，但 sentinel service 未正常啟動，也無法正確執行 failover，使用 `journalctl -xe` 檢查時出現下列錯誤訊息

    - 錯誤訊息內容

        ```txt
        Sentinel config file /etc/redis/redis_sentinel_26379.conf is not writable: Permission denied. Exiting...
        ```

    - 錯誤截圖

        ![3confignotwriteable](https://user-images.githubusercontent.com/3851540/74103982-7e510e00-4b8b-11ea-8d46-7483a0da3dc0.png)

    - 解決方式

        >grant sentinel config `寫入` 的權限給 service 的執行使用者(以 `redis` 為例)

        ```bash
        setfacl -m u:redis:rw /etc/redis/redis_sentinel_26379.conf
        ```

## 心得

過去安裝都是人工使用有 root 權限帳號進行安裝，幾乎沒遇到權限問題，透過這次使用 script 來進行遠端安裝，立馬讓過去權限不分的問題暴露出來XD

原本以為自己對 Redis 還算熟悉，沒想到過去使用時忽略了不少權限設定上的細節，資安觀念薄弱讓 server 使用過大的權限來執行 Redis 服務，所幸還沒發生資料外漏或是駭客攻擊，不過還是相當慚愧呀，算是在出問題前學到了寶貴一課

## 參考資訊

1. [使用 Yum 安裝 Redis 5](/yum-install-redis5)
2. [Redis: Creating Server TCP listening socket *:6388: bind: Permission denied](https://stackoverflow.com/questions/53009361/redis-creating-server-tcp-listening-socket-6388-bind-permission-denied)
3. [Redis: Failed opening .rdb for saving: Permission denied](https://stackoverflow.com/a/28686802/3600583)
