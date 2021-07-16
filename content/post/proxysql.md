---
title: "使用 ProxySQL 來簡化 MySQL 的讀寫分離"
date: 2021-07-15T21:30:00+08:00
lastmod: 2021-07-15T21:30:00+08:00
draft: false
tags: ["MySQL","ProxySQL"]
slug: "proxysql"
---

## 使用 ProxySQL 來簡化 MySQL 的讀寫分離

隨著系統使用者愈來愈多，對於 MySQL 的存取量也跟提高許多，為了增加系統整體 capacity，首先打算從 MySQL 著手調整起，在經過一連串指令的優化已經沒什麼調整空間後，便開始計劃加入 MySQL replication 機制，並使用讀寫分離的策略，雖然從程式面調整起，可能比較符合實際的需求，但整個調整的成本太高了，所以打算先評估 open source 工具：`ProxySQL`，今天簡單紀錄一下安裝與使用方式供同事 review

## 基本環境說明

1. macOS Big Sur 11.4
2. docker desktop 3.3.0 (62916)
3. docker images

    - bitnami/mysql:8.0
    - proxysql/proxysql:2.1.0

4. MySQL replication 與 ProxySQL 的安裝

    > 透過 [bitnami/bitnami-docker-mysql](https://github.com/bitnami/bitnami-docker-mysql) 提供的 yaml 加上 proxysql 來快速建立環境

    - proxysql.cnf

        > - 多給一組帳密 `radmin:radmin` 方便從 host 上連線至 proxysql；預設的 `admin:admin` 必需進到 proxysql 的 container 中使用，且需要自行安裝 mysql client

        > - 其中的 `monitor_username` 與 `monitor_password` 跟後續建立 monitor 用的 user 帳號密碼需相同，這邊以 `monitor:pass.123` 為例

        ```cnf
        datadir="/var/lib/proxysql"

        admin_variables=
        {
            admin_credentials="admin:admin;radmin:radmin"
            mysql_ifaces="0.0.0.0:6032"
        }
        
        mysql_variables=
        {
            threads=4
            max_connections=2048
            default_query_delay=0
            default_query_timeout=36000000
            have_compress=true
            poll_timeout=2000
            interfaces="0.0.0.0:6033"
            default_schema="information_schema"
            stacksize=1048576
            server_version="5.5.30"
            connect_timeout_server=3000
            monitor_username="monitor"
            monitor_password="pass.123"
            monitor_history=600000
            monitor_connect_interval=60000
            monitor_ping_interval=10000
            monitor_read_only_interval=1500
            monitor_read_only_timeout=500
            ping_interval_server_msec=120000
            ping_timeout_server=500
            commands_stats=true
            sessions_sort=true
            connect_retries_on_failure=10
        }
        ```

    - docker-compose.yml

        ```yaml
        version: '2.1'
        
        services:
        mysql-master:
            image: docker.io/bitnami/mysql:8.0
            ports:
            - '3306:3306'
            volumes:
            - 'mysql_master_data:/bitnami/mysql/data'
            environment:
            - MYSQL_REPLICATION_MODE=master
            - MYSQL_REPLICATION_USER=admin
            - MYSQL_REPLICATION_PASSWORD=pass.123
            - MYSQL_USER=yowko
            - MYSQL_PASSWORD=pass.123
            - MYSQL_DATABASE=test
            # ALLOW_EMPTY_PASSWORD is recommended only for development.
            #- ALLOW_EMPTY_PASSWORD=yes
            - MYSQL_ROOT_PASSWORD=pass.123
            healthcheck:
            test: ['CMD', '/opt/bitnami/scripts/mysql/healthcheck.sh']
            interval: 15s
            timeout: 5s
            retries: 6
        
        mysql-slave:
            image: docker.io/bitnami/mysql:8.0
            ports:
            - '3307:3307'
            depends_on:
            - mysql-master
            environment:
            - MYSQL_REPLICATION_MODE=slave
            - MYSQL_REPLICATION_USER=admin
            - MYSQL_REPLICATION_PASSWORD=pass.123
            - MYSQL_USER=yowko
            - MYSQL_PASSWORD=pass.123
            - MYSQL_DATABASE=test
            - MYSQL_MASTER_HOST=mysql-master
            - MYSQL_MASTER_PORT_NUMBER=3306
            - MYSQL_MASTER_ROOT_PASSWORD=pass.123
            # ALLOW_EMPTY_PASSWORD is recommended only for development.
            # - ALLOW_EMPTY_PASSWORD=yes
            healthcheck:
            test: ['CMD', '/opt/bitnami/scripts/mysql/healthcheck.sh']
            interval: 15s
            timeout: 5s
            retries: 6
        proxysql:
            image: proxysql/proxysql:2.1.0
            ports:
            - '6032:6032'
            - '6033:6033'
            - '6070:6070'
            volumes:
            - './proxysql.cnf:/etc/proxysql.cnf' 
            depends_on:
            - mysql-master
            - mysql-slave
        
        volumes:
        mysql_master_data:
            driver: local
        ```

5. 建立測試用 table 與資料

    > 用來驗證讀寫分離設定是否正確

    ```sql
    use test;
    create table testuser(
        userid INT NOT NULL AUTO_INCREMENT,
        username VARCHAR(100) NOT NULL,
        birthdate DATE,
        PRIMARY KEY ( userid )
    );  
    insert into testuser(username) values ('yowko');
    ```

## 設定方式

1. 新增並設定 ProxySQL 所需的 user

    > - 執行目標：`MySQL(master)`
    > - 連線方式：`mysql -h 127.0.0.1 -uroot -ppass.123`

    ![2connect](https://user-images.githubusercontent.com/3851540/125803417-613d7b43-e274-4f0f-9304-1a3a9505df05.png)

    - 使用 `mysql_native_password` 驗證機制新增 ProxySQL 所需的 user

        > ProxySQL 不支援 MySQL8 導入的 `caching_sha2_password` 驗證機式，必需指定使用舊驗證機制 `mysql_native_password`

        - `monitor`

            > 用來確認 MySQL 各個 server 是否正常服務用

            ```sql
            create user 'monitor'@'%' identified with mysql_native_password by 'pass.123';
            grant replication client on *.* to 'monitor'@'%';
            flush privileges;
            ```

        - `proxysql`

            > 用來執行實際 sql script

            ```sql
            create user 'proxysql'@'%' identified with mysql_native_password by 'pass.123';
            grant all privileges on *.* to 'proxysql'@'%';
            flush privileges;
            ```

    - 未指定使用 `mysql_native_password` 的錯誤訊息

        > 這個是從 ProxySQL container 中使用 `monitor` 連線 MySQL (`mysql -h mysql-master -umonitor -ppass.123`)所取得的錯誤

        ```txt
        ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory
        ```

    - 未指定使用 `mysql_native_password` 的錯誤截圖

        ![3accessdenied](https://user-images.githubusercontent.com/3851540/125803420-3d45afdb-5deb-4833-90f6-fb7c5f9d9707.png)

        ![4pluginerror](https://user-images.githubusercontent.com/3851540/125803421-2625e290-36d5-49da-bbc3-b9691151a895.png)

2. 設定 ProxySQL 中的 MySQL 讀寫角色分組

    > - 執行目標：`ProxySQL 管理服務` (6032)
    > - 連線方式：`mysql -h 127.0.0.1 -P 6032 -uradmin -pradmin` (預設 proxysql container 的帳密是 `admin:admin`，但僅限 container 中使用)

    ![1connect](https://user-images.githubusercontent.com/3851540/125803406-949f373c-bdcd-4eb8-9da5-9278d5b81894.png)

    - 新增組別

        > `writer_hostgroup` 和 `reader_hostgroup` 的設定值都要大於 0 且不能相同 (下面使用 `writer_hostgroup` : `10`；`reader_hostgroup` : `20` 為例)

        ```sql
        insert into mysql_replication_hostgroups ( writer_hostgroup, reader_hostgroup, comment) values (10,20,'proxy');
        ```

    - 將 MySQL server 加入各自的組別中

        ```sql
        insert into mysql_servers(hostgroup_id,hostname,port) values (10,'mysql-master',3306);
        insert into mysql_servers(hostgroup_id,hostname,port) values (20,'mysql-slave',3306);
        ```

    - 將設定值載入 runtime

        ```sql
        load mysql servers to runtime;
        ```

    - 將設定值儲存至 disk (optional)

        ```sql
        save mysql servers to disk;
        ```

    - 確認是否可以正確連線至各個 MySQL

        ```sql
        select * from mysql_server_ping_log limit 10;
        ```

        - 正常連線

            ![5pingok](https://user-images.githubusercontent.com/3851540/125803423-5462ad6e-ff80-4fa4-8fcf-855ae5b8f69a.png)

        - 無法連線

            ![3accessdenied](https://user-images.githubusercontent.com/3851540/125803420-3d45afdb-5deb-4833-90f6-fb7c5f9d9707.png)

3. 將 ProxySQL 連線 MySQL 用 user 加至 ProxySQL 中

    > - 執行目標：`ProxySQL 管理服務` (6032)
    > - 連線方式：`mysql -h 127.0.0.1 -P 6032 -uradmin -pradmin` (預設 proxysql container 的帳密是 `admin:admin`，但僅限 container 中使用)

    - 新增 ProxySQL 連線 MySQL 用 user；載入 runtime;儲存至 disk (optional)

        ```sql
        insert into mysql_users (username,password,default_hostgroup) values ('proxysql','pass.123',10);
        load mysql users to runtime;
        save mysql users to disk;
        ```

4. 建立讀寫分流規則

    > - 執行目標：`ProxySQL 管理服務` (6032)
    > - 連線方式：`mysql -h 127.0.0.1 -P 6032 -uradmin -pradmin` (預設 proxysql container 的帳密是 `admin:admin`，但僅限 container 中使用)

    - 新增讀寫分流規則;載入規則至 runtime;儲存規則至 disk (optional)

        > 以下規則表示 script 若是以 `select` 開頭就導向 destination_hostgroup 為 `20` 的 `reader_hostgroup`

        ```sql
        insert into mysql_query_rules(rule_id,active,match_pattern,destination_hostgroup,apply) values (2,1,'^select',20,1);
        load mysql query rules to runtime;
        save mysql query rules to disk;
        ```

5. 實際效果

    1. 新增並查詢資料

        > - 執行目標：`ProxySQL 對外服務` (6033)
        > - 連線方式：`mysql -uproxysql -ppass.123 -P 6033 -h 127.0.0.1`

        ```sql
        mysql -uproxysql -ppass.123 -P 6033 -h 127.0.0.1 -e "insert into test.testuser(username) values ('yowko');"
        mysql -uproxysql -ppass.123 -P 6033 -h 127.0.0.1 -e "select * from test.testuser;"
        ```

    2. 確認讀寫分離規則

        > - 執行目標：`ProxySQL 管理服務` (6032)
        > - 連線方式：`mysql -h 127.0.0.1 -P 6032 -uradmin -pradmin` (預設 proxysql container 的帳密是 `admin:admin`，但僅限 container 中使用)

        ```sql
        select hostgroup,schemaname,username,digest_text,count_star from  stats_mysql_query_digest;
        ```

        ![6rulecheck](https://user-images.githubusercontent.com/3851540/125803429-d9b0e27e-ba19-4077-b068-f678e1b1d531.png)

## 心得

以我粗淺的 db 知識來看，ProxySQL 架構滿複雜的，但厲害的是指令並不難理解，不過我怕我講錯所以今天的筆記絕口不提架構，請大家自行努力了，我就先求個會用、過渡一下，待 DBA 報到後應該就可以交接給 DBA 了

雖然我從架環境到實際完成讀寫分離，花了好幾天，但與逐一調整每個 application 程式碼相比還是相當節省時間的，不過有個隱憂是畢竟多了一層 proxy，實際的效能衰退狀況還不確定，待做個測試取得前後差異再來決擇吧

## 參考資訊

1. [bitnami/bitnami-docker-mysql](https://github.com/bitnami/bitnami-docker-mysql)
2. [ProxySQL](https://proxysql.com/)
3. [sysown/proxysql](https://github.com/sysown/proxysql)
4. [Why ERROR 1045 (28000): Access denied in ProxySQL server?](https://stackoverflow.com/a/64595396)
5. [ProxySQL 基礎篇](https://www.cnblogs.com/keme/p/12290977.html)
6. [ProxySQL 配置详解及读写分离(+GTID)等功能说明 (完整篇)](https://www.cnblogs.com/kevingrace/p/10329714.html)
7. [MySQL ProxySQL介绍](https://www.modb.pro/db/28841)
