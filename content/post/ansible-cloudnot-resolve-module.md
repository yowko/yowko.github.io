---
title: "Ansible 無法解析 module"
date: 2023-05-04T00:30:00+08:00
lastmod: 2023-05-04T00:30:31+08:00
draft: false
tags: ["ansible"]
slug: "ansible-cloudnot-resolve-module"
---

## Ansible 無法解析 module

這是前幾天使用 Helm 安裝 elastic filebeat 時遇到的狀況，除了 production 的其他環境都相當順利，原本  擔心 template 在 generate helm value file 時可能會因為 production 用的帳密，複雜度較高，會出現需要 escape 的情境，所以在真正執行安裝前先用 check mode 檢查一下，想不到 template generate 正常，反而是出現 ansible module 解析異常的錯誤，之前沒有遇過相關問題，找到處理方式後才覺得之前沒遇過反而奇怪，趁這個機會紀錄一下

## 基本環境設定

1. Linux Kernel 3.10.0-1160.36.2.el7.x86_64
2. CentOS 7
3. ansible 2.9.27
4. python 2.7.5 (default, Nov 16 2020, 22:23:17) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
5. ansible task

    ```yaml
    - name: Install filebeat with redis module
      kubernetes.core.helm:
        name: redisbeat
        chart_ref: elastic/filebeat
        chart_version: "{{filebeat_chart_version}}"
        release_namespace: "{{redisbeat_namespace}}"
        values: "{{ lookup('template', 'redisbeat.yml.j2') | from_yaml }}"
    ```

## 錯誤資訊

1. 無法解析 `kubernetes.core.helm`

    - 錯誤訊息

        ```txt
        fatal: [localhost]: FAILED! => {"reason": "couldn't resolve module/action 'kubernetes.core.helm'. This often indicates a misspelling, missing collection, or incorrect module path.\n\nThe error appears to be in '/home/vqzopdxbwntwtjv/it-automation-redisslowlog/roles/redis-slowlog/tasks/install.yml': line 4, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n    msg: \"{{ lookup('template', 'redisbeat.yml.j2') | from_yaml }}\"\n- name: Install filebeat with redis module\n  ^ here\n"}
        ```

    - 錯誤截圖

        ![1error1](https://user-images.githubusercontent.com/3851540/236398586-6d0b5340-df3c-407a-8169-95bd1712e4df.png)

2. 錯誤的參數

    > 這是收到無法解析的錯誤訊息後的嘗試：將 module name 改為 `helm`

    ```yaml
    - name: Install filebeat with redis module
      helm:
        name: redisbeat
        chart_ref: elastic/filebeat
        chart_version: "{{filebeat_chart_version}}"
        release_namespace: "{{redisbeat_namespace}}"
        values: "{{ lookup('template', 'redisbeat.yml.j2') | from_yaml }}"
    ```

    - 錯誤訊息

        ```txt
        fatal: [localhost]: FAILED! => {"changed": false, "msg": "Unsupported parameters for (helm) module: chart_ref, chart_version, release_namespace Supported parameters include: chart, disable_hooks, host, name, namespace, port, state, values"}
        ```

    - 錯誤截圖

        ![2error2](https://user-images.githubusercontent.com/3851540/236398542-91e61f09-27af-471c-84d1-295f9db776a8.png)

## 設定方式

- 透過列出已安裝的 collection，檢查是否安裝目標 collection

    - 使用 ansible-galaxy

        ```bash
        ansible-galaxy collection list
        ```

    - 直接列出安裝位置的內容

        ```bash
        ls ~/.ansible/collections/ansible_collections && ls /usr/share/ansible/collections/ansible_collections
        ```

- 安裝 Kubernetes Collection for Ansible.

    ```bash
    ansible-galaxy collection install kubernetes.core
    ```

## 心得

找到問題原因：未安裝 `kubernetes.core` collection 後，我更多疑問了：那其他環境怎麼會有？！我連怎麼檢查都是這次遇到問題才知道怎麼用的. 

後來懷疑到是不是版本的差異，對了一輪後發現只有 production 是 ansible 2.9.27，其他環境都是 2.11.* 以上，我沒有找到相關文件說明，但也只能推測可能是後續的版本有內建了

## 參考資訊

1. [kubernetes.core.helm module - Manages Kubernetes packages with the Helm package manager](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/helm_module.html)
2. [Using Ansible collections/Listing collections](https://docs.ansible.com/ansible/latest/collections_guide/collections_listing.html)
3. [Kubernetes Collection for Ansible](https://galaxy.ansible.com/kubernetes/core?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW)
