---
title: "使用 Ansible 安裝 InfluxDB 並建立 User，Database，Retention Policy"
date: 2020-03-01T15:30:00+08:00
lastmod: 2021-11-03T15:30:31+08:00
draft: false
tags: ["Ansible","InfluxDB"]
slug: "ansible-influxdb-create-user-database-rententionpolicy"
---

## 使用 Ansible 安裝 InfluxDB 並建立 User，Database，Retention Policy

之前筆記 [使用 Ansible 安裝 InfluxDB](/ansible-install-influxdb/) 紀錄到如何使用 Ansible 來安裝 InfluxDB，也提到透過 Ansible 安裝 InfluxDB 與直接執行 script 來安裝 差異不大，不過透過 Ansible 安裝 InfluxDB 好處主要是在於後續建立 User，Database，Retention Policy 上，所以立馬來紀錄一下囉

## 基本環境說明

1. Azure 標準 B1ms (1 vcpu，2 GiB 記憶體)
2. Centos 7.7
3. ansible 2.7.8
4. InfluxDB 1.7
5. ansible script

    - inventory.ini

        ```ini
        [influxdb]
        influx1 ansible_host=192.168.1.101 ansible_port=22 ip=192.168.1.101  ansible_user=yowko ansible_password=pass.123 ansible_become_password=pass.123
        ```

    - install.yml

        ```yaml
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
                  baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/    stable
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
        ```

## 個別語法

1. create user

    ```yml
    - name: Create a user
      shell: |
        curl -XPOST http://localhost:8086/query --data-urlencode "q=CREATE USER admin WITH PASSWORD pass.123 WITH ALL PRIVILEGES"
    ```

2. create database

    ```yml
    - name: Create Database
      shell: |
        curl -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE yowko"
    ```

3. create retention policy

    ```yml
    - name: Create Retention Policy
      shell: |
        curl -XPOST http://localhost:8086/query --data-urlencode "q=CREATE RETENTION POLICY yowko_test ON yowko DURATION 90d REPLICATION 1"
    ```

## 實際語法

1. 將可以設定的變數皆抽出來統一管理
2. 需要檢查 influxdb service 啟動與否

    > 詳細內容可以參考之前筆記 [Ansible 透過 Http Status Code 當做檢核條件](/ansible-when-http-status-code)

3. 需要設定啟用 auth
4. 完整語法

    ```yml
    ---
    - name: Install Influxdb
      gather_facts: false
      vars:
        influxdb_username: admin
        influxdb_password: pass.123
        influxdb_database_name: yowko
        influxdb_rp_name: yowko_test
        influxdb_url: http://localhost:8086
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
        - name: "wait for InfluxDB to come up"
          uri:
            url: "{{influxdb_url}}/ping"
            status_code: 204
          register: result
          until: result.status == 204
          retries: 60
          delay: 1
        - name: Create a user
          shell: |
            curl -XPOST {{influxdb_url}}/query --data-urlencode "q=CREATE USER {    {influxdb_username}} WITH PASSWORD '{{influxdb_password}}' WITH ALL PRIVILEGES"
        - name: Create Database
          shell: |
            curl -XPOST {{influxdb_url}}/query --data-urlencode "q=CREATE DATABASE {    {influxdb_database_name}}"
        - name: Create Retention Policy
          shell: |
            curl -XPOST {{influxdb_url}}/query --data-urlencode "q=CREATE RETENTION POLICY {    {influxdb_rp_name}} ON {{influxdb_database_name}} DURATION 90d REPLICATION 1"
        - name: Enable http auth
          shell: |
            sed -i 's/# auth-enabled = false/auth-enabled = true/g' /etc/influxdb/influxdb.conf
        - name: Restart influxdb Service
          service:
            name: influxdb
            state: restarte
    ```

## 心得

之前安裝測試時失敗了好幾次，主要是因為 InfluxDB 安裝後，服務啟動需要時間，Ansible 直接執行 http request 動作時會遇到 servive 尚未提供服務的問題，後來加上 InfluxDB 建立的 health check 確認之後就利完成本設定，最後修改 InfluxDB 的 auth 設定並重啟服務即可

## 參考資訊

1. [使用 Ansible 安裝 InfluxDB](/ansible-install-influxdb/)
2. [Ansible 透過 Http Status Code 當做檢核條件](/ansible-when-http-status-code)
3. [InfluxDB API reference](https://docs.influxdata.com/influxdb/v1.7/tools/api/)
