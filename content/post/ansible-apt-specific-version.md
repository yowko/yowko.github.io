---
title: "透過 Ansible 使用 APT 安裝指定版本套件"
date: 2023-07-17T00:30:00+08:00
lastmod: 2023-07-17T00:30:31+08:00
draft: false
tags: ["ansible","linux"]
slug: "ansible-apt-specific-version"
---

## 透過 Ansible 使用 APT 安裝指定版本套件

這是從之前筆記 [透過 Ansible 加入 GPG key 的 APT repository](/ansible-apt-repository-gpg) 延伸而來，將使用 ansible 搭配 apt 來安裝 erlang 與 RabbitMQ 的指定版本，主要流程與步驟是參考 [RabbitMQ 官網：Installing on Debian and Ubuntu](https://www.rabbitmq.com/install-debian.html) 來的

```bash
#!/bin/sh

sudo apt-get install curl gnupg apt-transport-https -y

## Team RabbitMQ's main signing key
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | sudo gpg --dearmor | sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null
## Community mirror of Cloudsmith: modern Erlang repository
curl -1sLf https://ppa1.novemberain.com/gpg.E495BB49CC4BBE5B.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg > /dev/null
## Community mirror of Cloudsmith: RabbitMQ repository
curl -1sLf https://ppa1.novemberain.com/gpg.9F4587F226208342.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/rabbitmq.9F4587F226208342.gpg > /dev/null

## Add apt repositories maintained by Team RabbitMQ
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
## Provides modern Erlang/OTP releases
##
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main

## Provides RabbitMQ
##
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
EOF

## Update package indices
sudo apt-get update -y
```

## 基本環境說明

1. Azure VM Standard B2s (2 vcpu，4 GiB 記憶體)
2. Linux (ubuntu 22.04)
3. erlang 25.3.2.2
4. RabbitMQ 3.11.19

## 安裝方式

1. Install needed packages

    詳細說明請參考 [透過 Ansible 加入 GPG key 的 APT repository](/ansible-apt-repository-gpg)

    ```yml
    - name: 1. Install needed packages
      become: true
      ansible.builtin.apt:
        pkg:
        - curl
        - gnupg
        - apt-transport-https
    ```

2. One way to avoid apt_key once it is removed from your distro

    詳細說明請參考 [透過 Ansible 加入 GPG key 的 APT repository](/ansible-apt-repository-gpg)

    ```yml
    - name: 2. One way to avoid apt_key once it is removed from your distro
      block:
        - name: 2-1. Add Erlang gpg key
          become: true
          apt_key:
            url: https://ppa1.novemberain.com/gpg.E495BB49CC4BBE5B.key
            state: present
          register: _add_apt_key
          until: _add_apt_key is succeeded
          retries: 5
          delay: 2
        - name: 2-2. Add RabbitMQ gpg key
          become: true
          apt_key:
            url: https://ppa1.novemberain.com/gpg.9F4587F226208342.key
            state: present
          register: _add_apt_rabbitmq_key
          until: _add_apt_rabbitmq_key is succeeded
          retries: 5
          delay: 2
        - name: 2-3. Add Erlang repository
          become: true
          ansible.builtin.apt_repository:
            repo: deb https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu {{ ansible_distribution_release }} main
            state: present
            filename: erlang-repo
        - name: 2-4. Add RabbitMQ repository
          become: true
          ansible.builtin.apt_repository:
            repo: deb https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu {{ ansible_distribution_release }} main
            state: present
            filename: rabbitmq-repo
    ```

3. Add Pin preference

    將特別 package 與 version 透過設定 preference 的方式固定，詳細內容可以參考之前筆記 [APT 安裝指定版本套件](/apt-specific-version)

    - prefernece.j2

        ```yml
        Package: erlang*
        Pin: version {{erlang_version}}
        Pin-Priority: 1000
        
        Package: rabbitmq*
        Pin: version {{rabbitmq_version}}
        Pin-Priority: 1000
        ```

    ```yml
    - name: 3. Add Pin preference
      become: true
      template: src=./prefernece.j2 dest=/etc/apt/preferences.d/erlang_rabbitmq
    ```

4. Update Cache and apply policy

    這個步驟我在某些 vm 上可以不用執行，但有些 vm 會有 policy 未套用的狀況，可以依實際需求調整

    ```yml
    - name: 4. Update Cache and apply policy
      become: true
      ansible.builtin.shell: apt update -y && apt policy
    ```

5. Install Erlang and RabbitMQ

    ```yml
    - name: 5. Install Erlang and RabbitMQ
      become: true
      apt:
        package:
          - "erlang-nox={{erlang_version}}"
          - "rabbitmq-server={{rabbitmq_version}}"
        state: present
        install_recommends: false
    ```

6. 完整內容

    ```yaml
    - hosts: azure_ubuntu
      vars:
        erlang_version: 1:25.3.2.2-1
        rabbitmq_version: 3.11.19-1
      tasks:
      - name: 1. Install needed packages
        become: true
        ansible.builtin.apt:
          pkg:
          - curl
          - gnupg
          - apt-transport-https
      - name: 2. One way to avoid apt_key once it is removed from your distro
        block:
          - name: 2-1. Add Erlang gpg key
            become: true
            apt_key:
              url: https://ppa1.novemberain.com/gpg.E495BB49CC4BBE5B.key
              state: present
            register: _add_apt_key
            until: _add_apt_key is succeeded
            retries: 5
            delay: 2
          - name: 2-2. Add RabbitMQ gpg key
            become: true
            apt_key:
              url: https://ppa1.novemberain.com/gpg.9F4587F226208342.key
              state: present
            register: _add_apt_rabbitmq_key
            until: _add_apt_rabbitmq_key is succeeded
            retries: 5
            delay: 2
          - name: 2-3. Add Erlang repository
            become: true
            ansible.builtin.apt_repository:
              repo: deb https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu {{ ansible_distribution_release }} main
              state: present
              filename: erlang-repo
          - name: 2-4. Add RabbitMQ repository
            become: true
            ansible.builtin.apt_repository:
              repo: deb https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu {{ ansible_distribution_release }} main
              state: present
              filename: rabbitmq-repo
      - name: 3. Add Pin preference
        become: true
        template: src=./prefernece.j2 dest=/etc/apt/preferences.d/erlang_rabbitmq
      - name: 4. Update Cache and apply policy
        become: true
        ansible.builtin.shell: apt update -y && apt policy
      - name: 5. Install Erlang and RabbitMQ
        become: true
        apt:
          package:
            - "erlang-nox={{erlang_version}}"
            - "rabbitmq-server={{rabbitmq_version}}"
          state: present
          install_recommends: false
    ```

## 心得

該踩的坑大部份都在之前的筆記中紀錄了，唯一讓我覺得奇怪的是 `apt policy` 這個在 ansible apt module 沒有找到相關參數指令，我原本以為這個需求很常見，後來測試過程中發現好像不一定要執行，後續又測試了幾次但沒有找出問題的規則，只能用 shell 先補上，秉持著多做沒錯的原則避免少做出問題

完整原始碼：[yowko/ansible-apt-rabbitmq](https://github.com/yowko/ansible-apt-rabbitmq)

## 參考資訊

1. [透過 Ansible 加入 GPG key 的 APT repository](/ansible-apt-repository-gpg)
2. [APT 安裝指定版本套件](/apt-specific-version)
3. [RabbitMQ 官網：Installing on Debian and Ubuntu](https://www.rabbitmq.com/install-debian.html)
4. [yowko/ansible-apt-rabbitmq](https://github.com/yowko/ansible-apt-rabbitmq)
