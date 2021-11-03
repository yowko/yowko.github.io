---
title: "在 Linux 中將 Redis 安裝成 Service - 以 CentOS 7 為例"
date: 2017-09-06T00:12:00+08:00
lastmod: 2021-11-03T00:12:50+08:00
draft: false
tags: ["Linux","Redis"]
slug: "centos7-redis-service"
aliases:
    - /2017/09/centos7-redis-service.html
---
## 在 Linux 中將 Redis 安裝成 Service - 以 CentOS 7 為例

Windows 環境中將 Redis 安裝成 service 的做法，曾經在 [Windows 環境如何設定 Redis Master-Slave 與 Sentinel](/2017/03/windows-redis-master-slave-sentinel.html) 介紹過，最近同事要架設 Linux 環境 Redis 時，問到該如何將 Redis 安裝為 Linux Service 讓 Redis 隨系統啟動自動啟動，避免系統異常重啟後 Redis 無人啟動讓服務中斷，趁這個機會紀錄一下自己常用的做法

## 將 Redis 安裝為 Linux Service

1. 下載、解壓縮、編譯 Redis

    ```bash
    wget http://download.redis.io/releases/redis-4.0.1.tar.gz
    tar xzf redis-4.0.1.tar.gz
    cd redis-4.0.1
    make
    ```

    > 如果想要更方便使用 redis 相關指令，會透過 `make install` 將執行檔安裝至 `/usr/local/bin` 中

2. 準備 Redis instance 用的 config

    > 這個步驟無論是不是安裝成 Service 都是必要的。不知道如何開始？！ 官網上有提供 Redis 各個版本的範例：[Redis configuration](https://redis.io/topics/config)

3. 安裝 Linux Service

    > make (compile) redis 後，在 `/utils/` 資料夾下有個 `install_server.sh` 檔可用來安裝 Linux Service

    * 執行 `install_server.sh`

        ```bash
        ./redis-4.0.1/utils/install_server.sh
        ```

    * 設定使用的 port (預設 6379)

        ```bash
        Please select the redis port for this instance: [6379]
        ```

    * 設定使用的 config 位置 (預設 `/etc/redis/6379.conf`)

        ```bash
        Please select the redis config file name [/etc/redis/6379.conf]
        ```

    * 設定 log 位置 (預設 `/var/log/redis_6379.log`)

        ```bash
        Please select the redis log file name [/var/log/redis_6379.log]
        ```

    * 設定資料儲存目錄 (預設 `/var/lib/redis/6379`)

        ```bash
        Please select the data directory for this instance [/var/lib/redis/6379]
        ```

    * 設定 redis 執行檔位置 (預設 `/usr/local/bin/redis-server`)

        ```bash
        Please select the redis executable path [/usr/local/bin/redis-server]
        ```

    * 確認設定是否正確

        ```config
        Selected config:
        Port           : 6379
        Config file    : /etc/6379.conf
        Log file       : /var/log/redis_6379.log
        Data dir       : /var/lib/redis/6379
        Executable     : /usr/local/bin/redis-server
        Cli Executable : /usr/local/bin/redis-cli
        Is this ok? Then press ENTER to go on or Ctrl-C to abort.
        ```

    * 安裝成功

        ```log
        Copied /tmp/6379.conf => /etc/init.d/redis_6379
        Installing service...
        Successfully added to chkconfig!
        Successfully added to runlevels 345!
        Starting Redis server...
        Installation successful!
        ```

        ![1install](https://user-images.githubusercontent.com/3851540/30070736-f9d7669c-9296-11e7-8524-0ab3dc5aedb7.png)

4. 確認安裝狀態
    * 列出所有非系統 service 再自行挑選出 `level 3` 是開啟狀態的(表示隨系統啟動)

        > `chkconfig --list`

        ![2chkconfiglist](https://user-images.githubusercontent.com/3851540/30070738-fa01516e-9296-11e7-9c01-3fb2e31c0f9c.png)

    * 直接過濾出隨系統啟動的非系統 service

        > * 英文：`chkconfig --list | grep 3:on`
        > * 中文：`chkconfig --list | grep 3:開啟`

        ![3chkconfig3](https://user-images.githubusercontent.com/3851540/30070741-fa2c01a2-9296-11e7-9b82-cbf6b3ced2ca.png)

## 解除安裝 Linux Service

* 直接刪除

    > `chkconfig --del redis_6379`

    ![4delservice](https://user-images.githubusercontent.com/3851540/30070739-fa1b6ac2-9296-11e7-9b29-d08ab201c7de.png)

* 設定不隨開機啟動

    > `chkconfig --level 3 redis_6379 off`

    ![5levele3off](https://user-images.githubusercontent.com/3851540/30070740-fa1cb990-9296-11e7-8eac-884363d36ada.png)

## 心得

這是 Redis 內建的設定 Service 工具，相較於自行處理啟動指令與 service 設定檔方便不少，自從使用這個方法後就再也沒查過文件看要打哪些指令跟設定什麼參數，還深怕指令跟參數打錯，簡單的方法最適合才記得久嘛 哈哈

## 參考資訊

1. [How to install and use Redis on Linux](https://discuss.pivotal.io/hc/en-us/articles/205308418-How-to-install-and-use-Redis-on-Linux)
2. [7 Linux chkconfig Command Examples – Add, Remove, View, Change Services](http://www.thegeekstuff.com/2011/06/chkconfig-examples)
3. [Windows 環境如何設定 Redis Master-Slave 與 Sentinel](/2017/03/windows-redis-master-slave-sentinel.html)
4. [Redis configuration](https://redis.io/topics/config)
