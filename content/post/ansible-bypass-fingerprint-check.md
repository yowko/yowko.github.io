---
title: "避免 Ansible 無法存取第一次登入的 Server"
date: 2020-04-25T21:30:00+08:00
lastmod: 2020-04-25T21:30:31+08:00
draft: false
tags: ["Ansible","Linux"]
slug: "ansible-bypass-fingerprint-check"
---

## 避免 Ansible 無法存取第一次登入的 Server

之前開始透過 Ansible 來安裝一組 server，原本想說腳本大致上也調得差不多了，結果新 server 一來，ansible playbook 所有 task 都沒執行成功，仔細看了錯誤訊息，發現是 ssh 檢查遠端 server 的 fingerprint，未嘗登入過的就需要先手動允許連線，但 server 數量眾多，一一手動允許登入不切實際也不符合成本效益，更有違自動化的原則，所以查了資料，簡單備忘一下

## 基本環境說明

1. macOS Catalina 10.15.4
2. Azure VM - CentOS 7.7 標準 B2s (2 vcpu，4 GiB 記憶體)

## 錯誤訊息

1. 訊息內容

    ```bash
    fatal: [server1]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host."}
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/80284177-7a2a9a80-874f-11ea-8abc-6ce758adf945.jpg)

- 直接使用 ssh 連線的提示

    ![2sshhint](https://user-images.githubusercontent.com/3851540/80284180-7d258b00-874f-11ea-9f93-043b9ba0753f.jpg)

## 設定方式

> 設定 `ansible.cfg` 加入以下設定

```txt
[defaults]
host_key_checking = False
```

## 心得

看到解決方式後  我立馬知道就是我自己沒有花時間搞清楚 Ansible 的設定方式，雖然我也知道略過官方文件直接動手不是個好做法：常常會以自己的理解弄出非正統的方式，但有時候就是沒那麼多時間耐住性子完整看完文件，可能就是因為我這個壞習慣才無法成為獨當一面的大師吧

## 參考資訊

1. [How to ignore ansible SSH authenticity checking?](https://stackoverflow.com/questions/32297456/how-to-ignore-ansible-ssh-authenticity-checking)
