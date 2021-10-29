---
title: "使用 sysbench 來取得 ProxySQL 效能差異"
date: 2021-07-16T12:30:00+08:00
lastmod: 2021-07-16T12:30:00+08:00
draft: false
tags: ["MySQL","ProxySQL","Benchmark"]
slug: "proxysql-benchmark-sysbench"
---

## 使用 sysbench 來取得 ProxySQL 效能差異

在之前筆記 [使用 ProxySQL 來簡化 MySQL 的讀寫分離](/proxysql) 提到需要進行壓力測試取得透過 ProxySQL 與直接存取 MySQL 的效能數據差來評估是否採用 ProxySQL

今天就來紀錄使用 sysbench 來進行 ProxySQL 與 MySQL 的效能差異比較

至於 sysbench，相信以我的理解可能只會誤導，還是請大家自行查閱 [akopytov/sysbench](https://github.com/akopytov/sysbench) 了

## 基本環境說明

1. macOS Big Sur 11.4
2. docker desktop 3.3.0 (62916)
3. sysbench 1.0.20

    ```bash
    brew install sysbench
    ```

4. docker images

    - bitnami/mysql:8.0
    - proxysql/proxysql:2.1.0

5. MySQL replication 與 ProxySQL 的安裝

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

6. ProxySQL 讀寫分離設定請參考之前筆記 [使用 ProxySQL 來簡化 MySQL 的讀寫分離](/proxysql)

## 執行測試

> `/usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua` 這是 macOS 上的腳本位置，不同 os 可能有所不同

1. 針對 MySQL

    - 前置準備

        ```bash
        sysbench /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
        --oltp-table-size=10000 \
        --mysql-table-engine=innodb \
        --oltp-tables-count=10  \
        --mysql-db=test \
        --mysql-user=proxysql \
        --mysql-password=pass.123 --mysql-port=3306 \
        --mysql-host=127.0.0.1 \
        --max-requests=0 \
        --time=10 \
        --report-interval=1 \
        --threads=10 \
        --oltp-point-selects=1 \
        --oltp-simple-ranges=0 \
        --oltp_sum_ranges=0 \
        --oltp_order_ranges=0 \
        --oltp_distinct_ranges=0 \
        --oltp-read-only=off \
        prepare
        ```

        ![1mysqlprepare](https://user-images.githubusercontent.com/3851540/125905638-fe656ff3-ecdf-4111-bb6f-9fc30bef351a.png)

    - 執行測試

        ```bash
        sysbench /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
        --oltp-table-size=10000 \
        --mysql-table-engine=innodb \
        --oltp-tables-count=10  \
        --mysql-db=test \
        --mysql-user=proxysql \
        --mysql-password=pass.123 --mysql-port=3306 \
        --mysql-host=127.0.0.1 \
        --max-requests=0 \
        --time=10 \
        --report-interval=1 \
        --threads=10 \
        --oltp-point-selects=1 \
        --oltp-simple-ranges=0 \
        --oltp_sum_ranges=0 \
        --oltp_order_ranges=0 \
        --oltp_distinct_ranges=0 \
        --oltp-read-only=off \
        run
        ```

        ![2mysqlrun](https://user-images.githubusercontent.com/3851540/125905646-7a850a39-5dab-4388-86dc-b97053a97ad9.png)

    - 清理資料

        ```bash
        sysbench /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
        --oltp-table-size=10000 \
        --mysql-table-engine=innodb \
        --oltp-tables-count=10  \
        --mysql-db=test \
        --mysql-user=proxysql \
        --mysql-password=pass.123 --mysql-port=3306 \
        --mysql-host=127.0.0.1 \
        --max-requests=0 \
        --time=10 \
        --report-interval=1 \
        --threads=10 \
        --oltp-point-selects=1 \
        --oltp-simple-ranges=0 \
        --oltp_sum_ranges=0 \
        --oltp_order_ranges=0 \
        --oltp_distinct_ranges=0 \
        --oltp-read-only=off \
        cleanup
        ```

        ![3mysqlcleanup](https://user-images.githubusercontent.com/3851540/125905650-97015752-f965-4ec9-8fce-8576e931ceb3.png)

2. 針對 ProxySQL

    - 前置準備

        ```bash
        sysbench /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
        --oltp-table-size=10000 \
        --mysql-table-engine=innodb \
        --oltp-tables-count=10  \
        --mysql-db=test \
        --mysql-user=proxysql \
        --mysql-password=pass.123 \
        --mysql-port=6033 \
        --mysql-host=127.0.0.1 \
        --max-requests=0 \
        --time=10 \
        --report-interval=1 \
        --threads=10 \
        --oltp-point-selects=1 \
        --oltp-simple-ranges=0 \
        --oltp_sum_ranges=0 \
        --oltp_order_ranges=0 \
        --oltp_distinct_ranges=0 \
        --oltp-read-only=off \
        prepare
        ```

        ![4proxysqlprepare](https://user-images.githubusercontent.com/3851540/125905655-b058e884-53ef-4e61-a3ec-d9220874cb41.png)

    - 執行測試

        ```bash
        sysbench /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
        --oltp-table-size=10000 \
        --mysql-table-engine=innodb \
        --oltp-tables-count=10  \
        --mysql-db=test \
        --mysql-user=proxysql \
        --mysql-password=pass.123 \
        --mysql-port=6033 \
        --mysql-host=127.0.0.1 \
        --max-requests=0 \
        --time=10 \
        --report-interval=1 \
        --threads=10 \
        --oltp-point-selects=1 \
        --oltp-simple-ranges=0 \
        --oltp_sum_ranges=0 \
        --oltp_order_ranges=0 \
        --oltp_distinct_ranges=0 \
        --oltp-read-only=off \
        run
        ```

        ![5proxysqlrun](https://user-images.githubusercontent.com/3851540/125905661-22fae6f1-8633-4196-a72c-4ced242c62d8.png)

    - 清理資料

        ```bash
        sysbench /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
        --oltp-table-size=10000 \
        --mysql-table-engine=innodb \
        --oltp-tables-count=10  \
        --mysql-db=test \
        --mysql-user=proxysql \
        --mysql-password=pass.123 \
        --mysql-port=6033 \
        --mysql-host=127.0.0.1 \
        --max-requests=0 \
        --time=10 \
        --report-interval=1 \
        --threads=10 \
        --oltp-point-selects=1 \
        --oltp-simple-ranges=0 \
        --oltp_sum_ranges=0 \
        --oltp_order_ranges=0 \
        --oltp_distinct_ranges=0 \
        --oltp-read-only=off \
        cleanup
        ```

        ![6proxysqlcleanup](https://user-images.githubusercontent.com/3851540/125905663-682c41de-5164-4f22-99c1-d384f0953226.png)

## 測試數據

針對 MySQL 與 ProxySQL 各執行三次測試，每次測試完都會執行 cleanup，詳細測試數據如下

|次數|MySQL|ProxySQL|
|---|---|---|
|1|![7mysql1](https://user-images.githubusercontent.com/3851540/125905665-46406a1a-e9a4-489e-9163-e97e58fcbc25.png)|![8proxysql1](https://user-images.githubusercontent.com/3851540/125905667-2c198174-91d1-49df-ad3c-b348f0e5dbfb.png)|
|2|![10mysql2](https://user-images.githubusercontent.com/3851540/125905671-0e19adbd-5b4d-47e4-8ffd-74a767f0966b.png)|![9proxysql2](https://user-images.githubusercontent.com/3851540/125905669-91afc036-05e1-4d35-b842-ca1ca39d5351.png)|
|3|![11mysql3](https://user-images.githubusercontent.com/3851540/125905673-0c7b86ca-47ae-4500-a451-c91e89e1edb0.png)|![12proxysql3](https://user-images.githubusercontent.com/3851540/125905674-945627d9-a943-4b62-bc17-bdd68eef2afe.png)|

## 心得

效能數據看起來是互有勝負 (MySQL 2勝 1敗)，平均來看應該 MySQL 數據比較好，雖然在預料之中，但卻沒想到 ProxySQL 也有一勝，這樣看來要不是測試不夠準確就是 ProxySQL 對於效能的影響不大，目前先紀錄使用與設定方式，待後續與同事討論再評估下一步

## 參考資訊

1. [使用 ProxySQL 來簡化 MySQL 的讀寫分離](/proxysql)
2. [akopytov/sysbench](https://github.com/akopytov/sysbench)
3. [MySQL benchmark tool - sysbench](https://blog.xuite.net/misgarlic/weblogic/56170203-MySQL+benchmark+tool+-+sysbench)
4. [性能测试--【MySQL】Sysbench 性能压测](https://www.huaweicloud.com/articles/4372f34838e28ac6f8952e56d7e39ad2.html)
