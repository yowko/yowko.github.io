---
title: "Ansible 使用 Here document (cat << EOF) 遇到的問題"
date: 2020-03-01T09:30:00+08:00
lastmod: 2020-12-11T09:30:31+08:00
draft: false
tags: ["Ansible"]
slug: "ansible-cat-eof"
---

## Ansible 使用 Here document (cat << EOF) 遇到的問題

這是在嘗試使用 Ansible 來輸出多行 config 時遇到的問題，實際例子可以參考 [在 CentOS 7 上安裝 Redis Replication (Redis 5)](/install-redis/) 其中要準備 redis.conf 的部份就有使用到 `Here document (cat << EOF)` 的技巧，原本想要延用相同方式在 Ansible 上執行，卻遇到問題，後來又再次在安裝 InfluxDB 時遇到相同狀況，所以快速筆記一下

## 基本環境說明

1. Azure 標準 B1ms (1 vcpu，2 GiB 記憶體)
2. Centos 7.7
3. ansible 2.7.8
4. ansible script
    
        ---
        - name: Install Influxdb
          gather_facts: false
          vars:
          hosts: influxdb
          tasks:
            - name: Add repo
              shell: |
                cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
                [influxdb]
                name = InfluxDB Repository - RHEL \$releasever
                baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
                enabled = 1
                gpgcheck = 1
                gpgkey = https://repos.influxdata.com/influxdb.key
                EOF

## 錯誤訊息

1. 訊息內容

    ```txt
    The full traceback is:
    WARNING: The below traceback may *not* be related to the actual failure.
      File "/tmp/ansible_yum_payload_itApQ9/__main__.py", line 1437, in ensure
        current_repos = my.repos.repos.keys()
      File "/usr/lib/python2.7/site-packages/yum/__init__.py", line 1071, in <lambda>
        repos = property(fget=lambda self: self._getRepos(),
      File "/usr/lib/python2.7/site-packages/yum/__init__.py", line 694, in _getRepos
        self.getReposFromConfig()
      File "/usr/lib/python2.7/site-packages/yum/__init__.py", line 567, in     getReposFromConfig
        self.getReposFromConfigFile(repofn, repo_age=thisrepo_age)
      File "/usr/lib/python2.7/site-packages/yum/__init__.py", line 465, in     getReposFromConfigFile
        raise Errors.ConfigError(exception2msg(e))
    
    fatal: [influx1]: FAILED! => {
        "ansible_facts": {
            "pkg_mgr": "yum"
        }, 
        "changed": false, 
        "invocation": {
            "module_args": {
                "allow_downgrade": false, 
                "autoremove": false, 
                "bugfix": false, 
                "conf_file": null, 
                "disable_excludes": null, 
                "disable_gpg_check": false, 
                "disable_plugin": [], 
                "disablerepo": [], 
                "download_only": false, 
                "enable_plugin": [], 
                "enablerepo": [], 
                "exclude": [], 
                "install_repoquery": true, 
                "installroot": "/", 
                "list": null, 
                "name": [
                    "influxdb"
                ], 
                "releasever": null, 
                "security": false, 
                "skip_broken": false, 
                "state": "latest", 
                "update_cache": false, 
                "update_only": false, 
                "use_backend": "auto", 
                "validate_certs": true
            }
        },
        "msg": "Error accessing repos: File contains parsing errors: file:///etc/yum.repos.d/    influxdb.repo\n\t[line  2]:  name = InfluxDB Repository - RHEL 7\n\n\t[line  3]:      baseurl = https://repos.influxdata.com/rhel/7/x86_64/stable\n\n\t[line  4]:  enabled     = 1\n\n\t[line  5]:  gpgcheck = 1\n\n\t[line  6]:  gpgkey = https://repos.influxdata.    com/influxdb.key\n\n\t[line  7]:  EOF\n"
    }
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/75618802-3764a000-5bae-11ea-8348-eb81ecd24691.png)

## 解決方式

> 多隔一層 `cmd` 指令

```yml
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
```

## 心得

我覺這問題 Ansible 給出的錯誤訊息不好理解問題發生原因，造成透過錯誤訊息來 google 時比較發散，不過反過來透過語法來找資料就馬上得到答案了

至於問題發生原因，我沒有細究，畢竟這是工具的語法特性，以我目前的水準只求會用、可以解決問題就很滿足了，等到有需要了解語法背後含義的那天再來細看囉

## 參考資訊

1. [How to do multiline shell script in Ansible](https://stackoverflow.com/a/40230416)
2. [ansible的shell模块使用cat命令--EOF结束文本输入问题](https://blog.csdn.net/qqhappy8/article/details/90579737)
3. [shell cat <<EOF command adding space in front of every line](https://github.com/ansible/ansible/issues/39137)