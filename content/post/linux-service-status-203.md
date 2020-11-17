---
title: "Linux Service Status 203 無法啟動"
date: 2020-11-05T21:30:00+08:00
lastmod: 2020-11-17T21:30:31+08:00
draft: false
tags: ["Linux","CentOS"]
slug: "linux-service-status-203"
---

## Linux Service Status 203 無法啟動

這是最近團隊在升級 zookeeper 及 kafka 時遇到的問題，不過仔細想想應該不是只有 zookeeper 會有問題，所以標題就不限定軟體名稱，避免日後查資料時反而誤會了，專注在 linux service 無法啟動且收到 exited status 為 203 的實際狀況做個紀錄

## 基本環境說明

1. Linux (centos 8.2.2004)
2. Linux Kernel 5.7.12-1.el8.elrepo.x86_64
3. Zookeeper 3.5.8

## 問題描述狀況

* service 無法成功啟動

    - 錯誤訊息 (service status)

        ```txt
        [root@SB-TSRV14 ~]# service zookeeper_2181 start
        Redirecting to /bin/systemctl start zookeeper_2181.service
        [root@SB-TSRV14 ~]# service zookeeper_2181 status
        Redirecting to /bin/systemctl status zookeeper_2181.service
        ● zookeeper_2181.service
           Loaded: loaded (/etc/systemd/system/zookeeper_2181.service; disabled; vendor preset: disabled)
           Active: failed (Result: exit-code) since Wed 2020-11-04 18:35:38 CST; 3s ago
          Process: 23821 ExecStart=/home/kafka_9092/bin/zookeeper-server-start.sh /home/kafka_9092/config/zookeeper.properties (code=exited, status=203/EXEC)
         Main PID: 23821 (code=exited, status=203/EXEC)

        11月 04 18:35:38 SB-TSRV14 systemd[1]: Started zookeeper_2181.service.
        11月 04 18:35:38 SB-TSRV14 systemd[1]: zookeeper_2181.service: Main process exited, code=exited, status=203/EXEC
        11月 04 18:35:38 SB-TSRV14 systemd[1]: zookeeper_2181.service: Failed with result 'exit-code'.
        ```

    - 錯誤截圖

        ![1servicestatus](https://user-images.githubusercontent.com/3851540/98203310-c06a7d00-1f6e-11eb-8ad8-f6b06bb011a2.png)

* 檢查 journalctl 沒有其他錯誤訊息

    - 錯誤訊息

        ```txt
        ----
        11月 04 18:08:24 SB-TSRV14 systemd[1]: Started zookeeper_2181.service.
        -- Subject: 單位 zookeeper_2181.service 啟動已結束
        -- Defined-By: systemd
        -- Support: https://access.redhat.com/support
        --
        -- 單位 zookeeper_2181.service 啟動已結束。
        --
        [Unit]
        -- 啟動結果為 done。
        11月 04 18:08:24 SB-TSRV14 systemd[1]: zookeeper_2181.service: Main process exited, code=exited, status=203/        EXEC
        11月 04 18:08:24 SB-TSRV14 systemd[1]: zookeeper_2181.service: Failed with result 'exit-code'.
        ```

    - 錯誤截圖

        ![2journalctl](https://user-images.githubusercontent.com/3851540/98203315-c2ccd700-1f6e-11eb-9fdc-e426844a5aca.png)

* 手動執行 service 中的啟動指令，可以正常運行

    ![3manualstart](https://user-images.githubusercontent.com/3851540/98203317-c3fe0400-1f6e-11eb-934e-69e725d30842.png)

## 問題確認

1. 檢查 SELinux 狀態

    > 如果為 `Enforcing` 代表 service 無法啟動的原因很有可能就是 SELinux 未關閉造成的

    ```bash
    getenforce
    ```

2. 暫時關閉 SELinux 進行確認

    > 將 SELinux 設為 `Permissive` (重新開機會重設)並嘗試重新啟動 service，若能順利啟動就確認問題是 SELinux 造成的

    ```bash
    setenforce 0
    ```

    ![4setenforce](https://user-images.githubusercontent.com/3851540/98203319-c52f3100-1f6e-11eb-8981-23cfb1be41ef.png)

3. 完全關閉 SELinux

    ```bash
    setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    ```

## 心得

這個問題發生當下花了我三、四個小時排除，後來是同事 google 到有類似症狀經過測試後才確認的，事後我一直在想為什麼會發生這個問題：

1. 安裝腳本只有 zookeeper 沒有關閉 SELinux
2. 沒有充份理解 SELinux 的影響以及症狀

所幸有了這次經歷，相信下次類似問題就可以更快找到方向了

## 參考資訊

1. [SELinux](https://wiki.centos.org/zh-tw/HowTos/SELinux)
