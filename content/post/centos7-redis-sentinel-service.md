---
title: "在 CentOS 7 上將 Redis sentinel 安裝成開機啟動的 service"
date: 2017-09-21T00:52:00+08:00
lastmod: 2021-11-03T00:52:12+08:00
draft: false
tags: ["Linux","Redis"]
slug: "centos7-redis-sentinel-service"
aliases:
    - /2017/09/centos-7-redis-sentinel-service.html
    - /2017/09/centos7-redis-sentinel-service/
---
## 在 CentOS 7 上將 Redis sentinel 安裝成開機啟動的 service

之前曾經在 [在 Linux 中將 Redis 安裝成 Service - 以 CentOS 7 為例](/2017/09/centos7-redis-service.html) 介紹過如何使用 redis 內建的 `install_server.sh` 執行檔將 redis instance 安裝成開機啟動的 service，也在 [Windows 環境如何設定 Redis Master-Slave 與 Sentinel](/2017/03/windows-redis-master-slave-sentinel.html) 介紹過如何在 Windows 環境中將 redis instance 及 sentinel 安裝為 windows service

最近公司正在 review redis 相關設定，才發現筆記中缺了 redis sentinel 在 CentOS 7 上相關設定的內容，剛好趁這個機會補上

## 準備 sentinel config

詳細內容可以參考 [Windows 環境如何設定 Redis Master-Slave 與 Sentinel](/2017/03/windows-redis-master-slave-sentinel.html)

* 建立 sentinel config

    ```bash
    vi /etc/redis/sentinel.conf
    ```

* 修改 config (以下僅示範用，請依實際情況調整)

    ```config
    port 26379
    daemonize yes
    sentinel monitor master 127.0.0.1 6379 1
    sentinel down-after-milliseconds master 3000
    sentinel failover-timeout master 18000
    sentinel parallel-syncs master 1
    ```

## 設定 Service 啟動 script

1. 將 redis 預設腳本複製至 `/etc/init.d/` 中

    ```bash
    cp ~/redis-4.0.1/utils/redis_init_script /etc/init.d/redis_sentinel
    ```

2. 調整啟動 script

    > `vi /etc/init.d/redis_sentinel`

    * 修改 `REDISPORT`

        > 一般慣例 sentinel 使用 `26379`，但只要不跟 redis instance 重複， port 並沒有實際限制

    * 修改 `EXEC`

        > 使用 `redis-sentinel` 來執行比較方便 (使用 `redis-server` 需要加上 `--sentinel` 參數)

    * 修改 `CONF`

        > 指定上面準備的 sentinel config 位置

    * 下方範例可直接使用

        ```bash
        #!/bin/sh
        #
        # Simple Redis init.d script conceived to work on Linux systems
        # as it does use of the /proc filesystem.
                    REDISPORT=26379
        EXEC=/usr/local/bin/redis-sentinel
        CLIEXEC=/usr/local/bin/redis-cli
                    PIDFILE=/var/run/redis_${REDISPORT}.pid
        CONF="/etc/redis/sentinel.conf"
                    case "$1" in
            start)
                if [ -f $PIDFILE ]
                then
                        echo "$PIDFILE exists, process is already running or crashed"
                else
                        echo "Starting Redis server..."
                        $EXEC $CONF
                fi
                ;;
            stop)
                if [ ! -f $PIDFILE ]
                then
                        echo "$PIDFILE does not exist, process is not running"
                else
                        PID=$(cat $PIDFILE)
                        echo "Stopping ..."
                        $CLIEXEC -p $REDISPORT shutdown
                        while [ -x /proc/${PID} ]
                        do
                            echo "Waiting for Redis to shutdown ..."
                            sleep 1
                        done
                        echo "Redis stopped"
                fi
                ;;
            *)
                echo "Please use start or stop as first argument"
                ;;
        esac
        ```

## 設定 systemd 的定義檔

1. 建立 `redis_sentinel.service` 定義檔

    ```bash
    vi /etc/systemd/system/redis_sentinel.service
    ```

2. 新增定義檔內容

    ```config
    [Unit] 
    Description=Redis Sentinel on port 26379
    
    [Service] 
    Type=forking
    ExecStart=/etc/init.d/redis_sentinel start
    ExecStop=/etc/init.d/redis_sentinel stop
    
    [Install]
    WantedBy=multi-user.target
    ```

## 啟動 service

1. 重新載入 systemd 中的 config

    ```bash
    systemctl daemon-reload
    ```

2. 開啟開機自動啟動

    ```bash
    systemctl enable redis_sentinel.service
    ```

3. 立即啟動 service

    ```bash
    systemctl start redis_sentinel.service
    ```

* systemctl start redis_sentinel.service

    > 如果啟動失敗可以看到詳細訊息

    ![1sysstatus](https://user-images.githubusercontent.com/3851540/30655896-dad51f08-9e64-11e7-842f-a3400a54ddaa.png)

* systemctl list-unit-files | grep -i redis_sentinel.service

    > 可以用來看 redis_sentinel.service 的狀態

  * 啟動前

    ![2disable](https://user-images.githubusercontent.com/3851540/30655893-dad02da4-9e64-11e7-879c-efc2e5a499c8.png)

  * 啟動後

    ![3enable](https://user-images.githubusercontent.com/3851540/30655895-dad10c42-9e64-11e7-8409-d86457001888.png)

* redis-cli -p 26379 ping

    > 確認是否有正確啟動，回應 `PONG` 即正確

    ![4pong](https://user-images.githubusercontent.com/3851540/30655894-dad0c9ee-9e64-11e7-8880-20d9e9a23585.png)

## 心得

sentinel 建立 service 的方式不像一般 redis instance 那麼方便，設定相對繁瑣些，但依上述步驟操作也還算容易，經過上述設定後就可以讓 sentinel 也能在 server 開機時自動執行，讓 redis HA 機制不會因為 server 重啟而失效

## 參考資訊

1. [在 Linux 中將 Redis 安裝成 Service - 以 CentOS 7 為例](/2017/09/centos7-redis-service.html)
2. [Windows 環境如何設定 Redis Master-Slave 與 Sentinel](/2017/03/windows-redis-master-slave-sentinel.html)
3. [Systemd 入門教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
4. [Enabling or disabling services using systemctl instead of chkconfig in RHEL 7 / CentOS 7 / Scientific Linux 7](http://microdevsys.com/wp/enabling-or-disabling-services-using-systemctl-instead-of-chkconfig-in-rhel-7-centos-7-scientific-linux-7/)
5. [How to Install Redis Server on CentOS 7](https://linoxide.com/storage/install-redis-server-centos-7/)
6. [在 CentOS 7 安裝 Redis（方法二）](https://dotblogs.com.tw/supershowwei/2016/02/02/112238)
