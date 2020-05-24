---
title: "使用 Ansible 安裝 Redis Replication"
date: 2020-02-14T21:30:00+08:00
lastmod: 2020-05-24T21:30:31+08:00
draft: false
tags: ["Linux","Redis","Ansible"]
slug: "ansible-install-redis"
---

## 使用 Ansible 安裝 Redis Replication

之前筆記 [在 CentOS 7 上安裝 Redis Replication (Redis 5)](https://blog.yowko.com/install-redis) 紀錄到使用單一 shell script 來安裝完整 Redis Replication (Master，Slave 與 Sentinell)，後來同事提到這類安裝最好是透過 Ansible 處理，所以我就來試試囉

原則上整個安裝流程與步驟是依照 [在 CentOS 7 上安裝 Redis Replication (Redis 5)](https://blog.yowko.com/install-redis) 用 Ansible 語法改寫而來的

另外這是我第一次寫 Ansible，用得不對的地方請大家指教

## 基本環境說明

1. Azure VM B1s (1 vcpu,2GiB memory) X 3
2. CentOS-based 7.7
3. Redis 5.0.7
4. epel-release.noarch 0:7-11
5. ius-release.noarch 0:2-1.el7.ius
6. ansible 2.9.3
7. python 2.7.5

## 安裝語法

1. 準備 redis config 與 service 的 template

    - master and slave config

        > 我用 `config.j2` 做範例

        ```yml
        dir /data/redis
        bind {{item.value.ip}}
        requirepass {{redispass}}
        {% if item.value.role == 'slave' %}
        replicaof {{item.value.masterip}} {{item.value.masterport}}
        {% endif %}
        masterauth {{redispass}}
        port {{item.value.port}}
        pidfile /var/run/redis_{{item.value.port}}.pid
        {{item.value.rdb}}
        rename-command KEYS ""
        maxclients 10000
        {{item.value.aof}}
        {{item.value.aofpolicy}}
        ```

    - sentine config

        > 我用 `sentinel.j2` 做範例

        ```yml
        bind {{item.value.ip}}
        port {{item.value.port}}

        {% for master in master_dic %}
        dir /data/redis
        sentinel monitor master_{{ loop.index }} {{master.value.ip}} {{master.value.port}} {{ 1 if sentinels|length < 2 else (sentinels|length)-2 }}
        sentinel auth-pass master_{{ loop.index }} {{redispass}}
        sentinel down-after-milliseconds master_{{ loop.index }}  3000
        sentinel parallel-syncs master_{{ loop.index }}  1
        sentinel failover-timeout master_{{ loop.index }}  18000
        {% endfor %}
        ```

    - service

        > 我用 `service.j2` 做範例

        ```yml
        [Unit]
        Description=Redis persistent key-value database
        After=network.target
        After=network-online.target
        Wants=network-online.target

        [Service]
        {% if item.value.role == 'sentinel' %}
        ExecStart=/usr/bin/redis-sentinel /etc/redis/redis_{{item.value.port}}.conf --supervised systemd
        {% else %}
        ExecStart=/usr/bin/redis-server /etc/redis/redis_{{item.value.port}}.conf --supervised systemd
        {% endif %}

        ExecStop=/usr/libexec/redis-shutdown
        Type=notify
        User=redis
        Group=redis
        RuntimeDirectory=redis_{{item.value.port}}
        RuntimeDirectoryMode=0755

        [Install]
        WantedBy=multi-user.target
        ```

2. 指定安裝的 host 與 data

    - inventory.ini
  
        ```yml
        [redis-nodes]
        node11 ansible_host=10.0.0.4  ip=10.0.0.4  ansible_user=root ansible_password=pass.123 ansible_become_password=pass.123
        node12 ansible_host=10.0.0.12 ip=10.0.0.12  ansible_user=root ansible_password=pass.123 ansible_become_password=pass.123
        ```

    - data.yml

        ```yml
        redispass: pass.123
        masters:
          redis1:
            ip: 10.0.0.4
            port: 6379
            role: master
            aof: appendonly no
            aofpolicy: appendfsync no
            rdb : |
              save ""
          redis2:
            ip: 10.0.0.12
            port: 6379
            role: master
            aof: appendonly no
            aofpolicy: appendfsync no
            rdb : |
              save ""
        slaves:
          redis1:
            ip: 10.0.0.12
            port: 6380
            role: slave
            masterip: 10.0.0.4
            masterport: 6379
            aof: appendonly no
            aofpolicy: appendfsync no
            rdb : |
              save ""
          redis2:
            ip: 10.0.0.4
            port: 6380
            role: slave
            masterip: 10.0.0.12
            masterport: 6379
            aof: appendonly no
            aofpolicy: appendfsync no
            rdb : |
              save ""
        sentinels:
          redis3:
            ip: 10.0.0.12
            port: 26379
            role: sentinel
          redis4:
            ip: 10.0.0.12
            port: 26380
            role: sentinel
          redis5:
            ip: 10.0.0.4
            port: 26379
            role: sentinel
        ```

3. 安裝腳本

    > 我用 `install.yml` 做範

    ```yml
        ---
        - name: Install Tools
          gather_facts: false
          hosts: all
          vars_files:
            - data.yml
          tasks:
            - name: Install IUS
              yum:
                name: https://centos7.iuscommunity.org/ius-release.rpm
                state: latest
            - name: Install Redis
              yum:
                name: redis5
                state: latest
            - name: Create folder for config
              file:
                path: /etc/redis
                state: directory
                mode: 0755
            - name: Create Folders
              shell: install -d -m 0755 -o redis -g redis /data /data/redis
            - name: Remove default redis service
              shell: |
                systemctl disable redis
                rm -rf /usr/lib/systemd/system/redis.service
                rm -rf /etc/systemd/system/redis.service.d
                rm -rf /etc/systemd/system/redis-sentinel.service.d
            - name: Disable SELinux
              shell: |
                setenforce 0
                sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
            - name: Allow Overcommit Memory
              sysctl:
                name: vm.overcommit_memory
                value: "1"
                state: present
                reload: yes
                ignoreerrors: yes

        - name: Prepare Configs
          hosts: all
          vars_files:
            - data.yml
          tasks:
            - name: Master and Slave config
              template:
                src: config.j2
                dest: "/etc/redis/redis_{{item.value.port}}.conf"
              when: ansible_host == item.value.ip
              with_items:
                - "{{ lookup('dict', masters,wantlist=True) }}"
                - "{{ lookup('dict', slaves,wantlist=True) }}"
            - name: Sentinel Config
              vars:
                master_dic: "{{ lookup('dict', masters,wantlist=True) }}"
              template:
                src: sentinel.j2
                dest: "/etc/redis/redis_{{item.value.port}}.conf"
                owner: redis
              when: ansible_host == item.value.ip
              loop: "{{ lookup('dict', sentinels,wantlist=True) }}"
            - name: Prepare Services
              template:
                src: service.j2
                dest: "/etc/systemd/system/redis_{{item.value.port}}.service"
              when: ansible_host == item.value.ip
              with_items:
                - "{{ lookup('dict', masters,wantlist=True) }}"
                - "{{ lookup('dict', slaves,wantlist=True) }}"
                - "{{ lookup('dict', sentinels,wantlist=True) }}"
        - name: Start Services
          hosts: all
          vars_files:
            - data.yml
          tasks:
            - name : Reload Service Setting
              shell: |
                systemctl daemon-reload
              when: ansible_host == item.value.ip
              with_items:
                - "{{ lookup('dict', masters,wantlist=True) }}"
                - "{{ lookup('dict', slaves,wantlist=True) }}"
                - "{{ lookup('dict', sentinels,wantlist=True) }}"
            - name: Restart Redis Service
              service:
                name: redis_{{item.value.port}}
                state: restarted
              when: ansible_host == item.value.ip
              with_items:
                - "{{ lookup('dict', masters,wantlist=True) }}"
                - "{{ lookup('dict', slaves,wantlist=True) }}"
                - "{{ lookup('dict', sentinels,wantlist=True) }}"
    ```

4. 卸載腳本

    > 我用 `uninstall.yml` 做範例

    ```yml
        ---
        - name: Uninstall Redis
          gather_facts: false
          hosts: all
          vars_files:
            - data.yml
          tasks:
            - name: uninstall redis
              shell: |
                systemctl stop redis_{{item.value.port}}
                systemctl disable redis_{{item.value.port}}
                rm -rf /etc/redis/redis_{{item.value.port}}.conf
                rm -rf /etc/systemd/system/redis_{{item.value.port}}.service
                systemctl daemon-reload
              when: ansible_host == item.value.ip
              with_items:
                - "{{ lookup('dict', masters,wantlist=True) }}"
                - "{{ lookup('dict', slaves,wantlist=True) }}"
                - "{{ lookup('dict', sentinels,wantlist=True) }}"
    ```

5. 使用方式

    - install

        ```bash
        ansible-playbook -i inventory.ini install.yml -vvv -b
        ```

    - uninstall

        ```bash
        ansible-playbook -i inventory.ini uninstall.yml -vvv -b
        ```

## 心得

整體來說，我個人是比較推薦使用 Ansible

- 優點：
  1. 語法看來比較整潔
  2. 彈性也很大

- 缺點：
  1. 不僅要熟悉 Ansible 的語法特性，
  2. 還有了解不少 module 的用途
  3. 加上我這次有用上的 Jinja2 語法要適應

我自己覺得有些進入門檻，如果排除用得好不好這個問題，其實也可以把本來的 script 直接塞進去 Ansible 執行，本來搞不定時我也想這麼做XD，但最後還是過不了自己那關

雖然我覺得還是有好幾段語法可以改用 Ansible module 或是加上執行前的條件確認，但初步來說已經達成目標，成果還算滿意，暫時就先這樣囉，有機會再來改善吧

完整原始碼在 [yowko/ansible-install-redis](https://github.com/yowko/ansible-install-redis)

## 參考資訊

1. [在 CentOS 7 上安裝 Redis Replication (Redis 5)](https://blog.yowko.com/install-redis)
2. [Ansible中文权威指南](https://ansible-tran.readthedocs.io/en/latest/index.html)
3. [yowko/ansible-install-redis](https://github.com/yowko/ansible-install-redis)
