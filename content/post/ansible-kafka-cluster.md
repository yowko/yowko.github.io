---
title: "Ansible 安裝 Kafka Cluster"
date: 2023-02-27T21:30:00+08:00
lastmod: 2023-02-27T21:30:31+08:00
draft: false
tags: ["Linux","Ansible","Kafka"]
slug: "ansible-kafka-cluster"
---

## Ansible 安裝 Kafka Cluster

之前筆記 [Ubuntu 安裝 Kafka KRaft cluster](/kafka-cluster-kraft-ubuntu/) 紀錄到在 Ubuntu 上安裝 KRaft mode (不使用 ZooKeeper) 的 Kafka cluster，雖然內容大致算清楚，但畢竟在切換不同 host 時需要自行調整，彈性不足，加上團隊也早就使用 Ansible 來管理安裝腳本，所以連帶紀錄一下 Ansible 安裝的腳本

## 基本環境說明

1. Azure VM B2s (2 vcpu,4GiB memory) x3

    - 10.1.0.4
    - 10.1.0.5
    - 10.1.0.6

2. Linux (ubuntu 22.04)
3. Kafka 3.4.0
4. OpenJDK 11 JDK
5. ansible 2.10.2

## Ansible 腳本

1. 專案結構
    - roles
        - kafka
            - defaults
                - main.yml
            - tasks
                - install.yml
                - main.yml
                - restart.yml
                - uninstall.yml
            - templates
                - kafka.j2
                - kafka.service.j2
            - vars
                - main.yml
    - inventories
      - dev.ini
    - ansible.cfg
    - main.yml

2. 完整內容

    - roles
        - kafka
            - defaults
                - main.yml

                  ```yml
                  ---
                  purpose: upgrade
                  kafka_replicationfactor: 3
                  ```

            - tasks
                - install.yml

                    ```yml
                    ---
                    - name: Install JDK
                      become: true
                      ansible.builtin.apt:
                        name: openjdk-11-jdk
                        state: present
                  
                    - name: Add user kafka
                      become: true
                      shell: |
                        groupadd --system kafka
                        useradd -s /sbin/nologin --system -g kafka kafka
                      ignore_errors: yes
                    
                    - name: Create kafka data dir
                      become: true
                      file:
                        path: "{{ item }}"
                        state: directory
                        mode: '1775'
                        owner: kafka
                        group: kafka
                        recurse: true
                      with_items:
                        - "{{kafka_home}}_{{kafka_port}}"
                        - "{{kafka_home}}_{{kafka_port}}/log"
                    
                    - name: Downloading kafka
                      become: true
                      get_url:
                        url: "{{ kafka_source_url }}"
                        dest: "{{kafka_home}}_{{kafka_port}}/kafka.tgz"
                        mode: '0755'
                    
                    - name: Unzip kafka
                      become: true
                      shell: tar -xvzf kafka.tgz --strip 1
                      args:
                        chdir: "{{kafka_home}}_{{kafka_port}}"
                    
                    - name: Config kafka for cluster & Setup server id
                      become: true
                      template: 
                        src: templates/kafka.j2
                        dest: "{{kafka_home}}_{{kafka_port}}/config/kraft/server.properties"
                        owner: kafka
                        group: kafka
                    
                    - name: create cluster id
                      become: true
                      run_once: true
                      shell: "{{kafka_home}}_{{kafka_port}}/bin/kafka-storage.sh random-uuid"
                      register: cluster_id
                    
                    - name: create get cluster id
                      run_once: true
                      set_fact:
                        kafka_cluster_id: "{{ cluster_id.stdout }}"
                    
                    - name: Create storage
                      become: true
                      shell: "{{kafka_home}}_{{kafka_port}}/bin/kafka-storage.sh format -t {{kafka_cluster_id}} -c {{kafka_home}}_{{kafka_port}}/config/kraft/server.properties"
                        
                    - name: Copy kafka's daemon config
                      become: true
                      template:
                        src:  templates/kafka.service.j2
                        dest: /etc/systemd/system/kafka_{{kafka_port}}.service
                        mode: '0755'
                        owner: kafka
                        group: kafka
                        backup: yes
                    
                    - name: Start kafka service
                      become: true
                      systemd:
                        name: kafka_{{kafka_port}}
                        state: started
                        enabled: yes
                        daemon_reload: yes
                  ```

                - main.yml

                    ```yml
                    ---
                    - name: Uninstall kafka
                      include_tasks: uninstall.yml
                      when: 
                        - purpose=='uninstall'
                    
                    - name: Install kafka
                      include_tasks: install.yml
                      when: 
                        - purpose=='install'
                        - inventory_hostname in groups['kafka_all']
                    
                    - name: Restart kafka
                      include_tasks: restart.yml
                      when: 
                        - purpose=='restart'
                        - inventory_hostname in groups['kafka_all']
                    ```

                - restart.yml

                    ```yml
                    ---
                    - name: restart kafka
                      become: true
                      systemd:
                        name: kafka
                        state: restarted
                        enabled: yes
                    ```

                - uninstall.yml

                    ```yml
                    ---
                    - name: Stop kafka service
                      become: true
                      systemd:
                        name: "{{item}}"
                        state: stopped
                        enabled: no
                      ignore_errors: yes
                      with_items: 
                        - kafka_{{kafka_port}}
                    
                    - name: Release kafka data dir
                      become: true
                      file:
                        path: "{{item}}"
                        state: absent
                      with_items: 
                        - "{{kafka_home}}_{{kafka_port}}"
                        - "/etc/systemd/system/kafka_{{kafka_port}}.service"
                    
                    - name: Systemd reread configs
                      become: true
                      systemd:
                        daemon_reload: yes
                    ```

            - templates
                - kafka.j2

                    ```j2
                    process.roles=broker,controller
                    broker.id={{ host_index | int +1}}
                    port={{kafka_port}}
                    
                    controller.quorum.voters={{voters}}
                    listeners=PLAINTEXT://:{{kafka_port}},CONTROLLER://:{{kafka_port|int+1}}
                    advertised.listeners=PLAINTEXT://{{ip}}:{{kafka_port}}
                    controller.listener.names=CONTROLLER
                    listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
                    
                    log.dirs={{kafka_home}}_{{kafka_port}}/log
                    ```

                - kafka.service.j2

                    ```j2
                    [Unit]
                    Description=Kafka
                    After=network.target
  
                    [Service]
                    Type=simple
                    User=kafka
                    ExecStart=/bin/sh -c '{{kafka_home}}_{{kafka_port}}/bin/kafka-server-start.sh {{kafka_home}}_{{kafka_port}}/config/kraft/server.properties > {{kafka_home}}_{{kafka_port}}/log/kafka.log 2>&1'
                    ExecStop={{kafka_home}}_{{kafka_port}}/bin/kafka-server-stop.sh
                    Restart=on-abnormal

                    [Install]
                    WantedBy=multi-user.target
                    ```

            - vars
                - main.yml

                    ```yml
                    ---
                    host_index: "{{groups['kafka_all'].index(inventory_hostname)}}"
                    voters: "{% set VOTER_ARR=[] %}{% for host in groups['kafka_all'] %}{% if   VOTER_ARR.insert(loop.index,(loop.index|string)+'@'+hostvars[host].ip+':'  +(hostvars[host].kafka_port|int+1)|string) %}{% endif %}{% endfor %}  {{VOTER_ARR|join(',')}}"
                    kafka_version: 3.4.0
                    kafka_partitions: 3
                    kafka_source_url: "https://downloads.apache.org/kafka/{{kafka_version}}/  kafka_2.13-{{kafka_version}}.tgz"
                    ```

    - inventories
        - dev.ini

            > `ansible_host` 是外網 ip 而 `ip` 則為內網 ip，如果 ansible client 是在內網，可以簡為 `ansible_host` 就好

            ```ini
            [all]
            kafka1 ansible_ssh_host=10.1.0.4 ansible_ssh_user=yowko ansible_ssh_pass=pass123 ip=10.1.0.4 ansible_become_password=pass.123
            kafka2 ansible_ssh_host=10.1.0.5 ansible_ssh_user=yowko ansible_ssh_pass=pass123 ip=10.1.0.5 ansible_become_password=pass.123
            kafka3 ansible_ssh_host=10.1.0.6 ansible_ssh_user=yowko ansible_ssh_pass=pass123 ip=10.1.0.6 ansible_become_password=pass.123
            
            [kafka_all]
            kafka1 kafka_port=9092 
            kafka2 kafka_port=9092 
            kafka3 kafka_port=9092 
            
            [kafka_all:vars]
            kafka_home="/home/kafka"
            ```

    - ansible.cfg

        ```conf
        [defaults]
        host_key_checking = False
        roles_path = ./roles
        ```

    - main.yml

      ```yaml
      - hosts: kafka_all
        any_errors_fatal: '{{ any_errors_fatal | default(true) }}'
        vars:
          purpose: 'install'
        roles:
          - kafka-cluster
      ```

3. 安裝

    ```bash
    ansible-playbook -i inventories/dev.ini main.yml
    ```

## 心得

大致上流程與之前筆記 [Ubuntu 安裝 Kafka KRaft cluster](/kafka-cluster-kraft-ubuntu/) 相同，主要是將原本的 script 改為 ansible 語法，另外原本 script 在處理多個 instance 較不彈性的部份也因為 ansible 特性的關係獲得有效解決

完整程式可以參考 [yowko/ansible-kafka](https://github.com/yowko/ansible-kafka)

## 參考資訊

1. [Ansible 安裝 Redis Cluster](/ansible-redis-cluster-update/)
2. [yowko/ansible-kafka](https://github.com/yowko/ansible-kafka)
