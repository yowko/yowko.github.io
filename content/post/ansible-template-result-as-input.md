---
title: "Ansible 直接將 template render 完成的結果作為 input"
date: 2023-02-22T00:30:00+08:00
lastmod: 2023-02-22T00:30:31+08:00
draft: false
tags: ["ansible"]
slug: "ansible-template-result-as-input"
---

## Ansible 直接將 template render 完成的結果作為 input

過去使用 ansible template 都是將 render 的結果暫存在某個資料夾中，需要套用的時候再指定這個暫存結果檔位置，雖然這樣的方式可以直接看到 render 完的結果內容，這樣的方式是方便 debug，不過缺點也是顯而易見的：

1. 增加無用檔案

    > 這個檔案是安裝用途的中介檔案，不需要常態性存在

2. 資料夾權限問題

    > 需要額外處理暫存資料夾與檔案的讀寫權限

3. 內容的更新時間不易控制

    > 如果內容不是每次都統一由 ansible render 出來，就容易出現直接改檔案而造成版控跟環境內容不一致

最近剛好新專案又遇到類似需求，google 看看有沒有其他做法，趁這個機會紀錄一下

## 基本環境說明

1. macOS Ventura 13.2
2. ansible 2.9.10
3. 使用 elastic/filebeat helm cahrt for redis 的 value template 做為範例

    ```Jinja2
    imageTag: 8.6.1
    daemonset:
      enabled: false
      extraEnvs: []
      secretMounts: []
    deployment:
      enabled: true
      extraEnvs: []
      secretMounts: []
      filebeatConfig:
        filebeat.yml: |
          filebeat.modules:
          - module: redis
            log:
              enabled: false
            slowlog:
              enabled: true
              var.hosts: [127.0.0.1:6379]
              var.password: ""
          output.elasticsearch:
            enabled: true
            hosts: ["http://127.0.0.1:9200"]
            #api_key: "id:api_key"
            #username: ""
            #password: ""
    ```

## 使用方式

1. 將 template render 成 file

    ```yaml
    - hosts: localhost
      tasks:
      - name: Test template
        template: src=./roles/test/templates/redisbeat.yml.j2 dest=testredisbeat.yaml
    ```

2. 直接使用 template render 的結果

    - 文字檔

        ```yaml
        - hosts: localhost
          tasks:
          - name: DEBUG
            debug:
              msg:  "{{ lookup('template', 'roles/test/templates/redisbeat.yml.j2') }}"
        ```

    - 轉為 yaml

        ```yaml
        - hosts: localhost
          tasks:
          - name: DEBUG
            debug:
              msg:  "{{ lookup('template', 'roles/test/templates/redisbeat.yml.j2') | from_yaml }}"
        ```

## 心得

這次紀錄的用法比較適合：以 template 產生的結果屬於不用真實存在的暫存性內容，像本文使用到的 helm value，只在安裝時需要提供值，安裝完成後就不再需要

但如果情境上是需要真正實體檔案，像是 linux service 相關定義內容或是某個 service 啟動所需要的 config file，這次紀錄的方式就不合適

## 參考資訊

1. [retrieve contents of file after templating with Jinja2](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_lookup.html)
