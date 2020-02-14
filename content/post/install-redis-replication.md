---
title: "在 CentOS 7 上安裝 Redis Replication (Redis 5)"
date: 2020-02-12T21:30:00+08:00
lastmod: 2020-02-12T21:30:31+08:00
draft: false
tags: ["Linux","Redis"]
slug: "install-redis"
---

## 在 CentOS 7 上安裝 Redis Replication (Redis 5)

之前筆記 [安裝 Redis 時遇到的錯誤排除](https://blog.yowko.com/install-redis-error) 紀錄到安裝 Redis Replication 時的幾個可能遇到的錯誤，經過數日小弟終於完成了初版可以一次安裝多個 Redis Replication 與 sentinel 的 shell script (雖說是初版，但這是我目前功力可以寫出的極限了XD，如果有大師指教才會調整)

我主要會用 yum 來安裝 Redis，如果對於 yum 安裝 redis 不熟悉可以參考之前筆記 [使用 Yum 安裝 Redis 5](https://blog.yowko.com/yum-install-redis5)

一直以來我就對 shell script 相當苦手，直到現在我還是搞不清楚 `()` 與 `(())`，`[]` 與 `[[]]` 到底差在哪兒，除此之外還有其他許多搞不清楚的地方：像是陣列的那些個功能、什麼時候變數應該要用 `""` 包....，不過因為功能比較重要，我就打算先記下來，有機會再請教同事

之前就發現同事的 shell script 寫得相當精美，剛好趁這次需求，仿照同事的 shell script 結構，偷學幾招，當然就要紀錄加深印象

## 基本環境說明

1. Azure VM B1s (1 vcpu,2GiB memory)
2. CentOS-based 7.7
3. Redis 5.0.7
4. epel-release.noarch 0:7-11
5. ius-release.noarch 0:2-1.el7.ius

## 安裝語法

```bash

main() {
    enter_parameters
    install_master
    install_slave
    install_sentinel
}

# 安裝 redis 與設定基本環境
install_tools()
    echo ">>> Install epel"
    sshpass -p pass.123 ssh root@$1 "yum install -y https://centos7.iuscommunity.orgius-release.rpm"

    echo ">>> Install redis"
    sshpass -p pass.123 ssh root@$1 "yum install -y redis5"
    sshpass -p pass.123 ssh root@$1 "mkdir -p /etc/redis && install -d -m 0755 -oredis -g redis /data /data/redis"


    # remove default redis service
    sshpass -p pass.123 ssh root@$1 "systemctl disable redis && rm -rf /usr/libsystemd/system/redis.service"

    # disable selinux
    sshpass -p pass.123 ssh root@$1 "setenforce 0 && sed -i 's/SELINUX=enforcingSELINUX=disabled/g' /etc/selinux/config"

    # overcommit memory setting to 1
    sshpass -p pass.123 ssh root@$1 "sysctl vm.overcommit_memory=1 && echo'vm.overcommit_memory = 1' >> /etc/sysctl.conf"
}

# 啟動 redis 服務
start_service()
{
    echo "Sttart Service @$1:$2"
    sshpass -p pass.123 ssh root@$1 "systemctl daemon-reload"
    sshpass -p pass.123 ssh root@$1 "systemctl enable redis_$2"
    sshpass -p pass.123 ssh root@$1 "systemctl start redis_$2"
}

install_master() {
    echo "###### Install Master ######"

    for index in "${!MASTER_IPs[@]}"; do
        # 安裝基本環境
        install_tools "${MASTER_IPs[$index]}"

        echo ">>> Prepare redis config"

cat <<EOF | sshpass -p pass.123 ssh root@${MASTER_IPs[$index]} "cat > /etc/redis/redis_${MASTER_PORTs[$index]}.conf"
dir /data/redis
bind 127.0.0.1 ${MASTER_IPs[$index]}
requirepass $PASSWORD
masterauth $PASSWORD
port ${MASTER_PORTs[$index]}
pidfile /var/run/redis_${MASTER_PORTs[$index]}.pid
save ""
rename-command KEYS ""
maxclients 10000
appendonly no
EOF

        echo ">>> Prepare redis service"

cat <<EOF | sshpass -p pass.123 ssh root@${MASTER_IPs[$index]} "cat > /etc/systemd/system/redis_${MASTER_PORTs[$index]}.service"
[Unit]
Description=Redis persistent key-value database
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis/redis_${MASTER_PORTs[$index]}.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis_${MASTER_PORTs[$index]}
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
EOF

    # Start Service
    start_service "${MASTER_IPs[$index]}" "${MASTER_PORTs[$index]}"

    done

}

install_slave() {
    echo "###### Install Slave ######"

    for index in "${!SLAVE_IPs[@]}"; do
        # 如果 ip 不在 master ip 清單中才執行安裝基本環境
        if [[ ! " ${MASTER_IPs[@]} " =~ " ${SLAVE_IPs[$index]} " ]]; then
            install_tools "${SLAVE_IPs[$index]}"
        fi

        echo ">>> Prepare redis config"

cat <<EOF | sshpass -p pass.123 ssh root@${SLAVE_IPs[$index]} "cat > /etc/redis/redis_${SLAVE_PORTs[$index]}.conf"
dir /data/redis
bind 127.0.0.1 ${SLAVE_IPs[$index]}
requirepass $PASSWORD
replicaof ${MASTER_IPs[$index]} ${MASTER_PORTs[$index]}
masterauth $PASSWORD
port ${SLAVE_PORTs[$index]}
pidfile /var/run/redis_${SLAVE_PORTs[$index]}.pid
save ""
rename-command KEYS ""
maxclients 10000
appendonly no
EOF

        echo ">>> Prepare redis service"
cat <<EOF | sshpass -p pass.123 ssh root@${SLAVE_IPs[$index]} "cat > /etc/systemd/system//redis_${SLAVE_PORTs[$index]}.service"
[Unit]
Description=Redis persistent key-value database
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis/redis_${SLAVE_PORTs[$index]}.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis_${SLAVE_PORTs[$index]}
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target


EOF

    # Start Service
    start_service "${SLAVE_IPs[$index]}" "${SLAVE_PORTs[$index]}"

    done
}

install_sentinel() {
    echo "###### Install Sentinel ######"

    for index in "${!SENTINEL_IPs[@]}"; do
        # 如果 ip 不在 master ip 與 slave ip 清單中才執行安裝基本環境
        if [[ ! " ${MASTER_IPs[@]} " =~ " ${SENTINEL_IPs[$index]} " || ! " ${SLAVE_IPs[@]} " =~ " ${SENTINEL_IPs[$index]} " ]]; then
            install_tools "${SENTINEL_IPs[$index]}"
        fi

        echo ">>> Prepare redis sentinel config"

cat <<EOF | sshpass -p pass.123 ssh root@${SENTINEL_IPs[$index]} "cat > /etc/redis/redis_${SENTINEL_PORTs[$index]}.conf"
bind 127.0.0.1 ${SENTINEL_IPs[$index]}
port ${SENTINEL_PORTs[$index]}
EOF
        # sentinel 可以 monitor 多個 master
        for masterIndex in "${!MASTER_IPs[@]}"; do

cat <<EOF | sshpass -p pass.123 ssh root@${SENTINEL_IPs[$index]} "cat >> /etc/redis/redis_${SENTINEL_PORTs[$index]}.conf"
dir /data/redis
sentinel monitor master_$masterIndex ${MASTER_IPs[$masterIndex]} ${MASTER_PORTs[$masterIndex]} $DEF_QUORUM
sentinel auth-pass master_$masterIndex $PASSWORD
sentinel down-after-milliseconds master_$masterIndex 3000
sentinel parallel-syncs master_$masterIndex 1
sentinel failover-timeout master_$masterIndex 18000
EOF
        done

        # 給 redis user 讀寫 sentinel config 權限
        sshpass -p pass.123 ssh root@${SENTINEL_IPs[$index]} "setfacl -m u:redis:rw /etc/redis/redis_${SENTINEL_PORTs[$index]}.conf"

        echo ">>> Prepare redis sentinel service"

cat <<EOF | sshpass -p pass.123 ssh root@${SENTINEL_IPs[$index]} "cat > /etc/systemd/system/redis_${SENTINEL_PORTs[$index]}.service"
[Unit]
Description=Redis persistent key-value database
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/redis-sentinel /etc/redis/redis_${SENTINEL_PORTs[$index]}.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis_${SENTINEL_PORTs[$index]}
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target


EOF

    # Start Service
    start_service "${SENTINEL_IPs[$index]}" "${SENTINEL_PORTs[$index]}"

    done
}


enter_parameters() {
    # 用來判斷是否終止輸入的預設值
    DEF_INPUT="-"
    # redis 預設密碼
    DEF_PASSWORD="pass.123"
    # sentinel 同意執行 failover 的個數
    DEF_QUORUM=1

    PASSWORD=$DEF_PASSWORD

    MASTER_ENDPOINTs=()
    MASTER_IPs=()
    MASTER_PORTs=()

    SLAVE_ENDPOINTs=()
    SLAVE_IPs=()
    SLAVE_PORTs=()

    SENTINEL_ENDPOINTs=()
    SENTINEL_IPs=()
    SENTINEL_PORTs=()

    while :
    do
        read -p "Redis Master Endpoint (ex:"127.0.0.1:6379" or press "Enter" to next step): " MASTER_ENDPOINT
        # 給定預設值
        MASTER_ENDPOINT=${MASTER_ENDPOINT:-$DEF_INPUT}
        # 收入的內容如果是預設值表示不再輸入
        if [ "$MASTER_ENDPOINT" == "$DEF_INPUT" ]; then
            break
        else
            # 輸入的 endpoint 沒有 `:` 就算輸入錯誤
            if [[ $MASTER_ENDPOINT != *":"* ]]; then
                echo "Wrong format"
                exit 1
            else
                MASTER_ENDPOINTs+=("$MASTER_ENDPOINT")
                # 拆解輸入的 endpoint
                IFS=':' read -ra ADDR <<< "$MASTER_ENDPOINT"
                index=0;
                for item in "${ADDR[@]}"; do
                    if (( index == 0 )); then
                        MASTER_IPs+=("$item")
                    else
                        MASTER_PORTs+=("$item")
                    fi
                    ((index=index+1))
                done
            fi
        fi
        # 結構與上面 master 相同
        read -p "Redis Slave Endpoint (ex:"127.0.0.1:6380" or press "Enter" to next step): " SLAVE_ENDPOINT
        SLAVE_ENDPOINT=${SLAVE_ENDPOINT:-$DEF_INPUT}
        if [ "$SLAVE_ENDPOINT" == "$DEF_INPUT" ]; then 
            break
        else
            if [[ $SLAVE_ENDPOINT != *":"* ]]; then
                echo "Wrong format"
                exit 1
            else
                SLAVE_ENDPOINTs+=("$SLAVE_ENDPOINT")
                IFS=':' read -ra ADDR <<< "$SLAVE_ENDPOINT"
                index=0;
                for item in "${ADDR[@]}"; do
                    if (( index == 0 )); then
                        SLAVE_IPs+=("$item")
                    else
                        SLAVE_PORTs+=("$item")
                    fi
                    ((index=index+1))
                done
            fi
        fi

    done

    # 結構與上面 master 相同，輸入完多組 master-slave 後再統一新增 sentinel
    while :
    do
        read -p "Redis Sentinel Endpoint (ex:"127.0.0.1:26379" or press "Enter" to next step): " SENTINEL_ENDPOINT
        SENTINEL_ENDPOINT=${SENTINEL_ENDPOINT:-$DEF_INPUT}
        if [ "$SENTINEL_ENDPOINT" == "$DEF_INPUT" ]; then 
            break
        else
            if [[ $SENTINEL_ENDPOINT != *":"* ]]; then
                echo "Wrong format"
                exit 1
            else
                SENTINEL_ENDPOINTs+=("$SENTINEL_ENDPOINT")
                IFS=':' read -ra ADDR <<< "$SENTINEL_ENDPOINT"
                index=0;
                for item in "${ADDR[@]}"; do
                    if (( index == 0 )); then
                        SENTINEL_IPs+=("$item")
                    else
                        SENTINEL_PORTs+=("$item")
                    fi
                    ((index=index+1))
                done
            fi
        fi

    done
    # 如果 sentinel 的個數大於 `1` 則投票同意數就改為 sentinel 個數-1
    if [[ "${#SENTINEL_ENDPOINTs[@]}" -gt "$DEF_QUORUM" ]]; then
        DEF_QUORUM="$((${#SENTINEL_ENDPOINTs[@]}-1))"
    fi

    echo "###### Parameters ######"
    echo "MASTER_ENDPOINTs=${MASTER_ENDPOINTs[*]}"
    echo "MASTER_IPs=${MASTER_IPs[*]}"
    echo "MASTER_PORTs=${MASTER_PORTs[*]}"
    echo "SLAVE_ENDPOINTs=${SLAVE_ENDPOINTs[*]}"
    echo "SLAVE_IPs=${SLAVE_IPs[*]}"
    echo "SLAVE_PORTs=${SLAVE_PORTs[*]}"
    echo "SENTTINEL_ENDPOINTs=${SENTINEL_ENDPOINTs[*]}"
    echo "SENTTINEL_IPs=${SENTINEL_IPs[*]}"
    echo "SENTTINEL_PORTs=${SENTINEL_PORTs[*]}"
    echo "DEF_QUORUM=$DEF_QUORUM"

    # master 與 slave 個數不符時，就代表某個 master 沒有對應的 slave，需要重新輸入
    if [[ "${#MASTER_ENDPOINTs[@]}" != "${#SLAVE_ENDPOINTs[@]}" ]]; then
        echo "redis master endpoints count doesn't match with slave"
        exit 1;
    fi
}

main "$@"
```

## 心得

寫完之後自我感覺良好呀，覺得我好像也會寫 shell script 了，哈哈

雖然有著同事的精美範例，但還是因為 [安裝 Redis 時遇到的錯誤排除](https://blog.yowko.com/install-redis-error) 的各種問題花了幾天的時間，語法的不熟悉也耗了不少時間，但終究是完成了一個可用版本，相當感謝同事

上面的 shell script 雖說可以符合我的使用情境，但個人覺得還是有些不足的地方：

1. config 與 service 不僅內容重複性高還散落在 `install_master`、`install_slave` 與 `install_sentinel` 中
2. 輸入 endpoint 的檢查與 ip、port 拆分或許也可以抽成共用 function

以上這兩點是我覺得有問題，有嘗試解決，但沒能花太多時間認真查解法 (需求有時間壓力，你懂的)，先記下來再讓各位大師指教了，當然有其他我沒想的也要麻煩各位大師了

完整程式碼請參考 [yowko/install-redis-replication](https://github.com/yowko/install-redis-replication)

## 參考資訊

1. [安裝 Redis 時遇到的錯誤排除](https://blog.yowko.com/install-redis-error)
2. [使用 Yum 安裝 Redis 5](https://blog.yowko.com/yum-install-redis5)
3. [yowko/install-redis-replication](https://github.com/yowko/install-redis-replication)
