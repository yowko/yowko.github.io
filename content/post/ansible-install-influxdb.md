---
title: "使用 Ansible 安裝 InfluxDB"
date: 2020-03-01T12:30:00+08:00
lastmod: 2020-12-11T12:30:31+08:00
draft: false
tags: ["Ansible","InfluxDB"]
slug: "ansible-install-influxdb"
---

## 使用 Ansible 安裝 InfluxDB

最近安裝實際 service instance 環境大多都用上了 Ansible，原本透過 Helm 安裝的 InfluxDB container 也想透過 Ansible 來建立實體 instance 不再使用 container，快速筆記一下語法

## 基本環境說明

1. Azure 標準 B1ms (1 vcpu，2 GiB 記憶體)
2. Centos 7.7
3. ansible 2.7.8
4. InfluxDB 1.7

## 安裝語法

1. inventory.ini

    ```ini
    [influxdb]
    influx1 ansible_host=192.168.1.101 ansible_port=22 ip=192.168.1.101  ansible_user=yowko ansible_password=pass.123 ansible_become_password=pass.123
    ```

2. install.yml

    > 其中有用到 Ansible 的 heredoc 技巧，詳細內容可以參考之前筆記 [Ansible 使用 Here document (cat << EOF) 遇到的問題](/ansible-cat-eof/)

        ---
        - name: Install Influxdb
          gather_facts: false
          hosts: influxdb
          tasks:
            - name: Add repo
              shell:
                cmd: |
                  cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
                  [influxdb]
                  name = InfluxDB Repository - RHEL \$releasever
                  baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
                  enabled = 1
                  gpgcheck = 1
                  gpgkey = https://repos.influxdata.com/influxdb.key
                  EOF
            - name: Install influxdb
              yum:
                name: influxdb
                state: latest
            - name: Start influxdb Service
              service:
                name: influxdb
                state: restarted

## 心得

原則上如果只是進行預設安裝，透過 Ansible 處理與直接執行 shell 差異不大，好處主要是在於便於統一管理安裝 script，透過一致的工具來執行，不過如果需要在安裝當下連帶建立 user、建立 daabase or 建立 retention policy，ansible 就會顯得簡潔許多

## 參考資訊

1. [Ansible 使用 Here document (cat << EOF) 遇到的問題](/ansible-cat-eof/)
2. [Installing InfluxDB OSS](https://docs.influxdata.com/influxdb/v1.7/introduction/installation/)
