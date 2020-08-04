---
title: "Ansible 安裝 Redis Replication 更新版"
date: 2020-07-04T21:30:00+08:00
lastmod: 2020-08-04T21:30:31+08:00
draft: false
tags: ["Ansible","Redis"]
slug: "ansible-redis-replication"
---

## Ansible 安裝 Redis Replication 更新版

之前筆記 [使用 Ansible 安裝 Redis Replication](https://blog.yowko.com/ansible-install-redis/) 中紀錄使用 Ansible 來安裝 Redis Replication，但畢竟是 Ansible 新手，很多好用的 function 都沒真的用上，多數都是土法鍊鋼用 script 硬上的，經過一段時間的練習，來更新一下自己覺得比較好的寫法，做為比較紀錄

## 基本環境說明

1. Azure VM B2s (2 vcpu,4GiB memory)
2. CentOS-based 7.7
3. Redis 5.0.7
4. epel-release.noarch 0:7-11
5. ius-release.noarch 0:2-1.el7.ius
6. ansible 2.9.10
7. python 2.7.15

## 安裝語法

1. 專案結構
    - roles
        - redis-replication
            - handlers
                - main.yml
            - tasks
                - checkdir.yml
                - installredis.yml
                - installtools.yml
                - main.yml
                - prepareservice.yml
                - prepareredisconfig.yml
                - preparesentinelconfig.yml
                - uninstall.yml
                - upgrade.yml
            - templates
                - config.j2
                - sentinel.j2
                - service.j2
            - vars
                - main.yml
            - inventories
                - dev.ini
            - README.md
    - ansible.cfg
    - main.yml

2. 完整內容

    - roles
        - redis-replication
            - handlers
                - main.yml

                  ```yml
                  ---
                  - name: "Restart Redis Service"
                    listen: restart-service
                    systemd:
                      name: "redis_{{redis_port}}"
                      daemon_reload: yes
                      enabled: yes
                      state: restarted
                  ```

            - tasks
                - checkdir.yml

                  ```yml
                  ---
                  - name: Ensures dir exists
                    file:
                      path: "{{ item.folder }}"
                      state: directory
                      mode: 0755
                      owner: redis
                      recurse: yes
                  ```

                - installredis.yml

                  ```yml
                  ---
                  - name: Install Redis
                    yum:
                      name: "{{redisversion}}"
                      state: latest
                  ```

                - installtools.yml

                  ```yml
                  - name: Install IUS
                    yum:
                      name:
                        - https://repo.ius.io/ius-release-el7.rpm
                        - https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
                      state: latest
                  
                  - include_tasks: installredis.yml
                  
                  - include_tasks: checkdir.yml
                    with_items:
                    - {folder: "/etc/redis"}
                  
                  - name: Remove default redis service
                    shell: |
                      systemctl disable redis
                  
                  - name: Delete redis service
                    file: 
                      path: "{{ item }}"
                      state: absent
                    with_items: 
                      - /usr/lib/systemd/system/redis.service
                      - /etc/systemd/system/redis.service.d
                      - /etc/systemd/system/redis-sentinel.service.d
                  
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
                  
                  - name: Stop and disable firewalld.
                    service:
                      name: firewalld
                      state: stopped
                      enabled: False
                  ```
                
                - main.yml
                
                  ```yml
                  ---
                  - include_tasks: uninstall.yml
                    when: action == 'uninstall' or action == 'reinstall'
                  
                  - include_tasks: installtools.yml
                    when: action == 'install'
                  
                  - include_tasks: upgrade.yml
                    when: action == 'upgrade'
                  
                  - include_tasks: prepareservice.yml
                    when: action == 'install' or action == 'reinstall'
                  
                  - include_tasks: prepareredisconfig.yml
                    when: action != 'uninstall'
                  
                  - include_tasks: preparesentinelconfig.yml
                    when: action != 'uninstall'
                  ```
                
                - prepareservice.yml
                
                  ```yml
                  ---
                  - name: Prepare services
                    template:
                      src: service.j2
                      dest: "/etc/systemd/system/redis_{{redis_port}}.service"
                      owner: redis
                  ```
                
                - prepareredisconfig.yml
                
                  ```yml
                  ---
                  - include_tasks: checkdir.yml
                    with_items: 
                    - {folder: "/etc/redis/redis_{{ redis_port }}"}
                  
                  - name: Prepare Master Configs
                    template:
                      src: config.j2
                      dest: "/etc/redis/redis_{{ redis_port }}.conf"
                      owner: redis
                    when: redis_role == "master" or redis_role == "slave"
                    notify: restart-service
                  ```
                
                - preparesentinelconfig.yml
                
                  ```yml
                  ---
                  - include_tasks: checkdir.yml
                    with_items: 
                    - {folder: "/etc/redis"}
                  
                  - name: Prepare Sentinel Configs
                    template:
                      src: sentinel.j2
                      dest: "/etc/redis/redis_{{redis_port}}.conf"
                      owner: redis
                    when: redis_role == "sentinel"
                    notify: restart-service
                  ```
                
                - uninstall.yml
                
                  ```yml
                  - name: Stop Service
                    service:
                      name: "redis_{{redis_port}}"
                      state: stopped
                  
                  - name: Disable Service
                    shell: |
                      systemctl disable redis_{{redis_port}}
                  
                  - name: Delete config
                    file:
                      path: "{{ item }}"
                      state: absent
                    with_items:
                      - /etc/redis/redis_{{redis_port}}.conf
                      - /etc/systemd/system/redis_{{redis_port}}.service

                  ```
                
                - upgrade.yml
            
                    ```yml
                    ---
                    - include_tasks: installredis.yml
                    ```

            - templates
                - config.j2
                
                    ```j2
                    dir /etc/redis/redis_{{redis_port}}
                    bind {{redis_ip}} 127.0.0.1
                    requirepass {{redispass}}
                    {% if redis_role == "slave" and master_target is defined %}
                    replicaof {{hostvars[master_target]['redis_ip']}} {{hostvars[master_target]['redis_port']                    }}
                    {% endif %}
                    masterauth {{redispass}}
                    port {{redis_port}}
                    pidfile /var/run/redis_{{redis_port}}.pid
                    maxclients 100000
                    maxmemory-policy volatile-lru
                    loglevel notice
                    ```
                
                - sentinel.j2
                
                    ```j2
                    bind {{redis_ip}}
                    port {{redis_port}}
                    
                    {% for master in groups['redis_masters'] %}
                    dir /etc/redis/redis_{{redis_port}}
                    
                    sentinel monitor master_{{ loop.index }} {{hostvars[master]['redis_ip']}} {{hostvars [master]['redis_port']}} {{ 1 if groups['redis_sentinels']|length < 2 else (groups                    ['redis_sentinels']|length)-2 }}
                    sentinel auth-pass master_{{ loop.index }} {{redispass}}
                    sentinel down-after-milliseconds master_{{ loop.index }}  3000
                    sentinel parallel-syncs master_{{ loop.index }}  1
                    sentinel failover-timeout master_{{ loop.index }}  18000
                    {% endfor %}
                    ```
                
                - service.j2

                    ```j2
                    [Unit]
                    Description=Redis persistent key-value database
                    After=network.target
                    After=network-online.target
                    Wants=network-online.target
                    
                    [Service]
                    {% if redis_role == 'sentinel' %}
                    ExecStart=/usr/bin/redis-sentinel /etc/redis/redis_{{redis_port}}.conf --supervised                     systemd
                    {% else %}
                    ExecStart=/usr/bin/redis-server /etc/redis/redis_{{redis_port}}.conf --supervised systemd
                    {% endif %}
                    
                    ExecStop=/usr/libexec/redis-shutdown
                    Type=notify
                    User=redis
                    Group=redis
                    RuntimeDirectory=redis_{{redis_port}}
                    RuntimeDirectoryMode=0755
                    
                    [Install]
                    WantedBy=multi-user.target
                    ```
                
            - vars
                - main.yml

                    ```yml
                    action: update
                    redispass: pass.123
                    redisversion: redis5-5.0.7-1.el7.ius
                    ```

            - inventories
                - dev.ini

                    ```ini
                    [redis_instances]
                    redis1 ansible_host=192.168.50.51 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123
                    redis2 ansible_host=192.168.50.51 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123
                    redis_sentinel_1 ansible_host=192.168.50.51 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123
                    redis_sentinel_2 ansible_host=192.168.50.51 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123
                    redis_sentinel_3 ansible_host=192.168.50.51 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123

                    [redis_masters]
                    redis1 redis_port=6379 redis_role=master

                    [redis_slaves]
                    redis2 redis_port=6380 redis_role=slave master_target=redis1

                    [redis_sentinels]
                    redis_sentinel_1 redis_port=26379 redis_role=sentinel
                    redis_sentinel_2 redis_port=26380 redis_role=sentinel
                    redis_sentinel_3 redis_port=26381 redis_role=sentinel

                    [redis_all:children]
                    redis_masters
                    redis_slaves
                    redis_sentinels
                    ```

            - README.md

                ```txt
                ## 全新安裝

                    ```bash
                    ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "action=install" main.yml
                    ```

                ## 重新安裝

                    > 先 uninstall 並排除安裝基本套件

                    ```bash
                    ansible-playbook -i roles/redis-replication/inventories/dev.ini -e                 "action=reinstall" main.yml
                    ```

                ## 更新 config

                    ```bash
                    ansible-playbook -i roles/redis-replication/inventories/dev.ini main.yml
                    ```

                ## 升級

                    ```
                    ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "action=upgrade" main.yml
                    ```
                
                ## restart service

                    ```
                    ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "action=restart" main.yml
                    ```
                ```

    - ansible.cfg

        ```conf
        [ironman]
        roles_path = ./roles

        [defaults]
        host_key_checking = False
        ```

    - main.yml

      ```yaml
      - hosts: redis_all
        roles:
          - redis-replication
      ```

## 心得

完整程式碼請參考 [ansible-redis-replication](https://github.com/yowko/ansible-redis-replication)

大致流程與之前筆記 [使用 Ansible 安裝 Redis Replication](https://blog.yowko.com/ansible-install-redis/) 相去不遠，只是改用 ansible 的 function 來處理

另外比較大的變化是使用 ansible role 的功能來管理，讓 ansible 的 script 可以擁有較高的共用性

## 參考資訊

1. [使用 Ansible 安裝 Redis Replication](https://blog.yowko.com/ansible-install-redis/)
2. [ansible-redis-replication](https://github.com/yowko/ansible-redis-replication)