---
title: "找不到 rabbitmqadmin ？！"
date: 2021-08-14T21:30:00+08:00
lastmod: 2021-08-14T21:30:00+08:00
draft: false
tags: ["CentOS","RabbitMQ"]
slug: "rabbitmqadmin-not-found"
---

## 找不到 rabbitmqadmin ？！

rabbimq 安裝完正要建立 queue 才發現 rabbitmqadmin 指令，google 了發現很簡單的方法可以解決，立馬筆記一下供日後備查

- 找不到 rabbitmqadmin

    ![1rabbitmqnotfound](https://user-images.githubusercontent.com/3851540/129543481-01625aba-ad86-49ad-94eb-dbb4e20f9e57.png)

## 基本環境說明

1. Azure VM : 標準 B2s (2 個 vcpu，4 GiB 記憶體)
2. OS image : CentOS 7.9 Free - Gen1
3. RabbitMQ 3.3.5

## 設定方式

1. 安裝 management plugin 並重啟 rabbitmq

    ```bash
    sudo rabbitmq-plugins enable rabbitmq_management && sudo systemctl restart rabbitmq-server
    ```

2. 從 management 下載 rabbitmqadmin

    ```bash
    curl -O http://127.0.0.1:15672/cli/rabbitmqadmin
    ```

3. 給予執行權限並移至 `/usr/local/sbin`

    ```bash
    sudo chmod +x rabbitmqadmin && sudo mv rabbitmqadmin /usr/local/sbin
    ```

## 心得

- 實際效果

    ![2rabbitmq](https://user-images.githubusercontent.com/3851540/129543488-3458a6a7-ce51-4957-b49b-4fcb94420eb3.png)

透過 rabbitmq management 來下載 rabbitmqadmin 滿方便的，不用從其他位置下載也不需要自行手動 build，缺點是需要安裝 rabbitmq_management，如果對這個步驟有疑慮也可以在下載完後再手動移除

```bash
sudo rabbitmq-plugins disable rabbitmq_management && sudo systemctl restart rabbitmq-server
```

## 參考資訊

1. [RabbitMQ creating queues and bindings from command line](https://stackoverflow.com/a/25915568)
