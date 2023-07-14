---
title: "透過 Ansible 加入 GPG key 的 APT repository"
date: 2023-07-14T00:30:00+08:00
lastmod: 2023-07-14T00:30:31+08:00
draft: false
tags: ["ansible","linux"]
slug: "ansible-apt-repository-gpg"
---

## 透過 Ansible 加入 GPG key 的 APT repository

最近正在將過去寫過的 Ansible 腳本改寫：由 CentOS 轉為 Ubuntu，因為 package management 工具不同，所以主要就是這部份改動較大，其他安裝流程大致沒變，除此之外最常見的就是 package name 在不同 package management tool 中不同，今天就來紀錄一下如何透過 Ansible 加入 GPG key 的 APT repository

## 基本環境說明

1. Azure VM Standard B2s (2 vcpu，4 GiB 記憶體)
2. Linux (ubuntu 22.04)

## 設定方式

流程是從 [RabbitMQ 官網：Installing on Debian and Ubuntu](https://www.rabbitmq.com/install-debian.html) 上抄來的

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

1. 安裝必要套件

    ```yml
    - name: Install needed packages
      become: true
      ansible.builtin.apt:
        pkg:
        - curl
        - gnupg
        - apt-transport-https
    ```

2. 加入 Erlang gpg key

    ```yml
    - name: Add Erlang gpg key
      become: true
      apt_key:
        url: https://ppa1.novemberain.com/gpg.E495BB49CC4BBE5B.key
        state: present
        validate_certs: false
      register: _add_apt_key
      until: _add_apt_key is succeeded
      retries: 5
      delay: 2
    ```

3. 設定 Erlang repository

    > 這個最重要的是移除 signed-by 部份 `[signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg]`

    ```yml
    - name: Add Erlang repository
      become: true
      ansible.builtin.apt_repository:
        repo: deb https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu {{ ansible_distribution_release }} main
        state: present
        filename: erlang-repo
    ```

4. 完整 ansible playbook

    ```yml
    - hosts: azure_ubuntu
      tasks:
      - name: Install needed packages
        become: true
        ansible.builtin.apt:
          pkg:
          - curl
          - gnupg
          - apt-transport-https
      - name: One way to avoid apt_key once it is removed from your distro
        block:
          - name: Add Erlang gpg key
            become: true
            apt_key:
              url: https://ppa1.novemberain.com/gpg.E495BB49CC4BBE5B.key
              state: present
            register: _add_apt_key
            until: _add_apt_key is succeeded
            retries: 5
            delay: 2
          - name: Add Erlang repository
            become: true
            ansible.builtin.apt_repository:
              repo: deb https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu {{ ansible_distribution_release }} main
              state: present
              filename: erlang-repo
    ```

## 心得

試了幾次 ansible 官網的做法：[ansible.builtin.apt_repository module - Add and remove APT repositories](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_repository_module.html)  一直沒有成功，不知道是 gpg key 沒有成功塞進去還是本來就不能使用 signed-by 的關係，最後來參考 [rockandska/ansible-role-erlang/tasks/install_Debian.yml](https://github.com/rockandska/ansible-role-erlang/blob/master/tasks/install_Debian.yml) 的方式，直接透過 `apt_key` module 來匯入 key 並不使用 signed-by 才成功，至於為什麼，目前還不清楚，以後有機會搞懂了再補充

- 執行前 (azure repo)

    ![1before](https://github.com/yowko/picsbed/assets/3851540/d02bdeff-e1d9-446c-9db3-aaea0b458e39)

- 執行後 (rabbitmq 官網建議的 repo)

    ![2after](https://github.com/yowko/picsbed/assets/3851540/c67d32bc-cc2f-4776-b444-45bf422487df)

## 參考資訊

1. [RabbitMQ 官網：Installing on Debian and Ubuntu](https://www.rabbitmq.com/install-debian.html)
2. [rockandska/ansible-role-erlang/tasks/install_Debian.yml](https://github.com/rockandska/ansible-role-erlang/blob/master/tasks/install_Debian.yml)
3. [ansible.builtin.apt module - Manages apt-packages](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
4. [ansible.builtin.apt_key module - Add or remove an apt key](https://github.com/rockandska/ansible-role-erlang/blob/master/tasks/install_Debian.yml)
5. [ansible.builtin.apt_repository module - Add and remove APT repositories](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_repository_module.html)
6. [How can I get a list of all repositories and PPAs from the command line into an install script?](https://askubuntu.com/questions/148932/how-can-i-get-a-list-of-all-repositories-and-ppas-from-the-command-line-into-an)
7. [How can I remove gpg key that I added using apt-key add -?](https://askubuntu.com/questions/107177/how-can-i-remove-gpg-key-that-i-added-using-apt-key-add)
