---
title: "Ansible 安裝 Redis Cluster (更新版)"
date: 2020-08-16T21:30:00+08:00
lastmod: 2020-08-16T21:30:31+08:00
draft: false
tags: ["Ansible","Redis"]
slug: "ansible-redis-cluster-update"
---

## Ansible 安裝 Redis Cluster

之前筆記 [Ansible 安裝 Redis Cluster](https://blog.yowko.com/ansible-redis-cluster) 紀錄了以 ansible 內建 function 為主的 redis cluster 安裝 script，最近因為需要將 redis5 更新至 redis6，重新 review 了 script，做了些調整與優化，紀錄一下

## 基本環境說明

1. Azure VM B2s (2 vcpu,4GiB memory)
2. OpenLogic 7.7
3. Redis 6.0.6
4. ansible 2.9.11

## 安裝語法

1. 專案結構
    - roles
        - redis-cluster
            - defaults
                - main.yml
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
        - redis-cluster
            - defaults
                - main.yml

                  ```yml
                  ---
                  purpose: update
                  ```

            - handlers
                - main.yml

                  ```yml
                  ---
                  - name: "Restart Redis Service"
                    listen: restart-service
                    systemd:
                      name: "redis_{{item}}"
                      daemon_reload: yes
                      enabled: yes
                      state: restarted
                    with_items: "{{redis_ports}}"
                  ```

            - tasks
                - checkdir.yml

                  ```yml
                  ---
                  - name: Ensures dir exists
                    file: 
                      path: "{{ folder }}"
                      state: directory
                      mode: 0755
                      owner: redis
                      group: redis
                      recurse: yes
                  ```

                - createcluster.yml

                  ```yaml
                  ---
                  - name: Create Cluster debug
                    debug:
                      msg: redis-cli -a {{redispass}} --cluster-replicas 1 --cluster create {% for redis_node in groups['redis_all'] %} {% for redis_port in hostvars[redis_node].redis_ports %} {{ hostvars[redis_node].ip }}:{{ redis_port }} {% endfor %}{% endfor %}

                  - name: Create Cluster
                    shell: |
                      echo "yes" |redis-cli -a {{redispass}} --cluster-replicas 1 --cluster create {% for redis_node in groups['redis_all'] %} {% for redis_port in hostvars[redis_node].redis_ports %} {{ hostvars[redis_node].ip }}:{{ redis_port }} {% endfor %}{% endfor %}
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
                  - name: Get rpm
                    shell: wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

                  - name: Install rpm
                    ignore_errors: yes
                    shell: rpm -Uvh remi-release-7.rpm

                  - name: Enable remi repo
                    shell: sed -i 's/enabled=0/enabled=1/g' /etc/yum.repos.d/remi.repo

                  - name: Insall Redis
                    include_tasks: installredis.yml

                  - name: check dir
                    include_tasks: checkdir.yml
                    vars:
                      folder: "/etc/redis"

                  - name: Remove default redis service
                    ignore_errors: yes
                    shell: |
                      systemctl disable redis 

                  - name: Delete redis service
                    ignore_errors: yes
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
                  - name: Uninstall
                    include_tasks: uninstall.yml
                    when: purpose == 'uninstall' or purpose == 'reinstall'

                  - name: Install rpm and redis
                    include_tasks: installtools.yml
                    when: purpose == 'install'

                  - name: Upgrde redis
                    include_tasks: upgrade.yml
                    when: purpose == 'upgrade'

                  - name: create linux user
                    shell: |
                      groupadd --system redis
                      useradd -s /sbin/nologin --system -g redis redis
                    ignore_errors: yes
                    when: purpose == 'install'

                  - name: Generate redis config file
                    include_tasks: prepareredisconfig.yml
                    when: purpose != 'uninstall'

                  - name: Generate service file
                    include_tasks: prepareservice.yml
                    when: purpose == 'install' or purpose == 'reinstall'

                  - name: Restart Service
                    debug:
                      msg: restart service
                    notify: restart-service
                    changed_when: true
                    when: purpose == 'restart'

                  - meta: flush_handlers

                  - name: Run cluster creation script
                    include_tasks: createcluster.yml
                    run_once: true
                    when: purpose == 'install' or purpose == 'reinstall'
                  ```

                - prepareredisconfig.yml

                  ```yml
                  ---
                  - name: check dir - /etc/redis/redis_{{ item }}
                    include_tasks: checkdir.yml
                    vars:
                      folder: "/etc/redis/redis_{{ item }}"
                    with_items: "{{ redis_ports }}"

                  - name: Prepare Configs
                    vars:
                      redis_port: "{{item}}"
                    template:
                      src: config.j2
                      dest: "/etc/redis/redis_{{ item }}.conf"
                      owner: redis
                    notify: restart-service
                    with_items:  "{{redis_ports}}"
                  ```

                - prepareservice.yml

                  ```yml
                  ---
                  - name: Prepare services
                    vars:
                      redis_port: "{{item}}"
                    template:
                      src: service.j2
                      dest: "/etc/systemd/system/redis_{{item}}.service"
                      owner: redis
                      group: redis
                    with_items: "{{redis_ports}}"
                  ```

                - uninstall.yml

                  ```yml
                  - name: Stop Service
                      service:
                        name: "redis_{{item}}"
                        state: stopped
                      with_items: "{{ redis_ports }}"

                    - name: Disable Service
                      shell: systemctl disable redis_{{item}}
                      with_items: "{{ redis_ports }}"

                    - name: Delete config
                      file:
                        path: "/etc/redis/redis_{{item}}.conf"
                        state: absent
                      with_items: "{{ redis_ports }}"

                    - name: Delete service
                      file:
                        path: "/etc/systemd/system/redis_{{item}}.service"
                        state: absent
                      with_items: "{{ redis_ports }}"

                    - name: Recursively remove directory
                      file:
                        path: /etc/redis/redis_{{item}}
                        state: absent
                      with_items: "{{ redis_ports }}"

                    - name: Remove directory
                      file:
                        path: /etc/redis/
                        state: absent
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

                  ExecStop=/usr/libexec/redis-shutdown redis/redis_{{redis_port}}
                  Type=notify
                  User=redis
                  Group=redis
                  RuntimeDirectory=redis_{{redis_port}}
                  RuntimeDirectoryMode=0755
                  LimitNOFILE=10000000
                  LimitCORE=infinity
                  LimitNPROC=infinity
                  LimitMEMLOCK=infinity

                  [Install]
                  WantedBy=multi-user.target
                  ```

            - vars
                - main.yml

                  ```yml
                  redispass: pass.123
                  redisversion: redis-6.0.6-1.el7.remi
                  ```

            - inventories
                - dev.ini

                  ```ini
                  [redis_all]
                  redis1 ansible_host=10.0.1.5 redis_ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_ports=7000,7003
                  redis2 ansible_host=10.0.1.6 redis_ip=10.0.1.6  ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_ports=7001,7004
                  redis3 ansible_host=10.0.1.7 redis_ip=10.0.1.7 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_ports=7002,7005
                  ```

            - README.md

              ```txt
              ## 全新安裝

                  ```bash
                  ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "purpose=install" main.yml
                  ```

              ## 重新安裝

                  > 先 uninstall 並排除安裝基本套件

                  ```bash
                  ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "purpose=reinstall" main.yml
                  ```

              ## 更新 config

                  ```bash
                  ansible-playbook -i roles/redis-replication/inventories/dev.ini main.yml
                  ```

              ## 升級

                  ```
                  ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "purpose=upgrade" main.yml
                  ```
              ## restart service

                  ```
                  ansible-playbook -i roles/redis-replication/inventories/dev.ini -e "purpose=restart" main.yml
                  ```
              ```

    - ansible.cfg

        ```conf
        [defaults]
        host_key_checking = False
        roles_path = ./roles
        ```

    - main.yml

      ```yaml
      - hosts: redis_all
        roles:
          - redis-cluster
      ```

## 心得

完整程式碼請參考 [ansible-redis-cluster](https://github.com/yowko/ansible-redis-cluster)

此次修改重點如下：

1. Redis 版本從 `5.0.7` 升級為 `6.0.6`
2. 控制流程變數由 `action` 改用 `purpose` (`action` 是保留字)
3. `defaults` 放變數預設值；`vars` 放常數值
4. `redis_port` 改 `redis_port` 可以用來處理單台 host 多個 redis instance，效能較好

## 參考資訊

1. [Ansible 安裝 Redis Cluster](https://blog.yowko.com/ansible-redis-cluster)
2. [ansible-redis-cluster](https://github.com/yowko/ansible-redis-cluster)
