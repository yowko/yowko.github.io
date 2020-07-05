---
title: "Ansible 安裝 Redis Cluster"
date: 2020-07-05T21:30:00+08:00
lastmod: 2020-07-05T21:30:31+08:00
draft: false
tags: ["Ansible","Redis"]
slug: "ansible-redis-cluster"
---

## Ansible 安裝 Redis Cluster

之前筆記 [Ansible 安裝 Redis Replication 更新版](https://blog.yowko.com/ansible-redis-replication) 紀錄了以 ansible 內建 function 為主的 redis replication 安裝 script，順手紀錄一下 redis cluster 的安裝方式

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
                - createcluster.yml
                - installredis.yml
                - installtools.yml
                - main.yml
                - prepareservice.yml
                - prepareredisconfig.yml
                - uninstall.yml
                - upgrade.yml
            - templates
                - config.j2
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
                - createcluster.yml

                    ```yaml
                    ---
                    - name: Create Cluster debug
                      debug:
                        msg: redis-cli -a {{redispass}} --cluster-replicas 1 --cluster create {% for redis_node in groups['redis_all'] %}{{ hostvars[redis_node]['redis_ip'] }}:{{ hostvars[redis_node]['redis_port'] }} {% endfor %}

                    - name: Create Cluster
                      shell: |
                        echo "yes" |redis-cli -a {{redispass}} --cluster-replicas 1 --cluster create {% for redis_node in groups['redis_all'] %}{{ hostvars[redis_node]['redis_ip'] }}:{{ hostvars[redis_node]['redis_port'] }} {% endfor %}
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
                    
                    - name: Restart Service
                      debug:
                        msg: restart service
                      notify: restart-service
                      changed_when: true
                      when: action == 'restart'
                    
                    - meta: flush_handlers
                    
                    - include_tasks: createcluster.yml
                      run_once: true
                      when: action == 'install' or action == 'reinstall'
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
                    masterauth {{redispass}}
                    port {{redis_port}}
                    pidfile /var/run/redis_{{redis_port}}.pid
                    maxclients 100000
                    # 啟用 redis cluster
                    cluster-enabled yes
                    # 每個 node 需要獨立，cluster 自行維護使用，不需人為介入
                    cluster-config-file nodes_{{redis_port}}.conf
                    # node 判斷失效的時間
                    cluster-node-timeout 5000
                    maxmemory-policy volatile-lru
                    loglevel notice
                    ```
                

                - service.j2

                    ```j2
                    [Unit]
                    Description=Redis persistent key-value database
                    After=network.target
                    After=network-online.target
                    Wants=network-online.target
                    
                    [Service]
                    ExecStart=/usr/bin/redis-server /etc/redis/redis_{{redis_port}}.conf --supervised systemd
                    
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
                    [redis_all]
                    redis1 ansible_host=127.0.0.1 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_port=7000
                    redis2 ansible_host=127.0.0.1 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_port=7001
                    redis3 ansible_host=127.0.0.1 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_port=7002
                    redis4 ansible_host=127.0.0.1 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_port=7003
                    redis5 ansible_host=127.0.0.1 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_port=7004
                    redis6 ansible_host=127.0.0.1 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_port=7005
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
            - redis-cluster
        ```

## 心得

完整程式碼請參考 [ansible-redis-cluster](https://github.com/yowko/ansible-redis-cluster)

安裝方式跟 redis-replication 一樣，而流程甚至更簡單：不用自行管理 master、slave 與 sentinel，不過中間還是有些需要注意的重點

1. 建立 cluster 前需要確保所有 redis service 都已啟動 (flush_handlers)
2. 另外是 cluster 的建立行為只需要執行一次 (`run_once: true`)

## 參考資訊

1. [Ansible 安裝 Redis Replication 更新版](https://blog.yowko.com/ansible-redis-replication)
2. [ansible-redis-cluster](https://github.com/yowko/ansible-redis-cluster)
