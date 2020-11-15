---
title: "Ansible 使用 dnf 安裝 Redis Cluster (Redis6)"
date: 2020-11-14T21:30:00+08:00
lastmod: 2020-11-14T21:30:31+08:00
draft: false
tags: ["Linux","Ansible","Redis"]
slug: "ansible-dnf-install-redis-cluster-redis6"
---

## Ansible 使用 dnf 安裝 Redis Cluster (Redis6)

之前筆記 [Ansible 安裝 Redis Cluster](https://blog.yowko.com/ansible-redis-cluster-update/) 紀錄到在 CentOS 7 上透過 Ansible 使用 yum 來安裝 Redis Cluster，但最近 production 環境已經升級成 CentOS 8 + dnf，所以來更新一下 Ansible script，避免之後想抄又要東翻西找XD

## 基本環境說明

1. Azure VM B2s (2 vcpu,4GiB memory)
2. OpenLogic CentOS 8.2
3. Redis 6.0.8
4. ansible 2.10.2

## Ansible 腳本

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
                    dnf:
                      name: "{{redisversion}}"
                      enablerepo:  remi
                      state: latest
                  ```

                - installtools.yml

                  ```yml
                  - name: Install rpm
                    dnf:
                      name: 'https://rpms.remirepo.net/enterprise/remi-release-8.rpm'
                      state: present
                      disable_gpg_check: yes

                  - name: Enable remi repo
                    shell: dnf module enable -y redis:remi-6.0

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
                    when: purpose != 'uninstall'

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
                  bind {{ip}} 127.0.0.1
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
                  redisversion: redis-0:6.0.8-1.el8.remi.x86_64
                  ```

            - inventories
                - dev.ini

                  > `ansible_host` 是外網 ip 而 `ip` 則為內網 ip，如果 ansible client 是在內網，可以精簡為 `ansible_host` 就好

                  ```ini
                  [redis_all]
                  redis1 ansible_host=10.0.1.5 ip=10.0.1.5 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_ports=7000,7003
                  redis2 ansible_host=10.0.1.6 ip=10.0.1.6  ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_ports=7001,7004
                  redis3 ansible_host=10.0.1.7 ip=10.0.1.7 ansible_ssh_user=root ansible_ssh_pass=pass.123 redis_ports=7002,7005
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

大致上流程與之前筆記 [Ansible 安裝 Redis Cluster](https://blog.yowko.com/ansible-redis-cluster-update/) 相同，主要變動的部份是

1. dnf module 的註冊與啟用
2. 使用 dnf 的 module 進行安裝

完整程式可以參考 [yowko/ansible-dnf-install-redis-cluster](https://github.com/yowko/ansible-dnf-install-redis-cluster)

## 參考資訊

1. [Ansible 安裝 Redis Cluster](https://blog.yowko.com/ansible-redis-cluster-update/)
2. [yowko/ansible-dnf-install-redis-cluster](https://github.com/yowko/ansible-dnf-install-redis-cluster)
