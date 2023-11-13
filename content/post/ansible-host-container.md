---
title: "使用 Container 做為 Ansible Host"
date: 2023-10-24T00:30:00+08:00
lastmod: 2023-10-24T00:30:31+08:00
draft: false
tags: ["docker","ansible"]
slug: "ansible-host-container"
---

## 使用 Container 做為 Ansible Host

過去在開發 ansible 腳本時為了做完整測試，都是在雲端建立 VM 來進行測試，但是這樣的方式有幾個缺點：

1. 建立 VM 需要花費時間
2. VM 的建立沒有額外的 source control，不易確保每次的環境都是一致的
3. VM 防火牆需要單獨 allow 特定 ip，但 ip 是會變動的
4. 網路連線會有延遲
5. 需要額外的費用

因此打算 container 來取代 VM 來當作 ansible host，這樣可以解決上述的問題，並且可以更快速的進行測試，其實過去也嘗試過，但可能是因為關鍵字下錯，所以找不到相關的資料，所以今天就來快速記錄一下。

今天會以為 RabbitMQ 建立 user 為範例

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 1.0.1(16297)
3. docker images
   - rabbitmq:3.12.7-management
   - yowko/rabbitmq:3.12.7-management
4. ansible [core 2.15.5]
5. python version = 3.11.6 (main, Oct  2 2023, 20:46:14) Clang 14.0.3 (clang-1403.0.22.14.1)
6. jinja version = 3.1.2
7. RabbitMQ cluster docker compose

    > 詳細內容可以參考過去筆記 [透過 docker compose 啟動 RabbitMQ cluster](/docker-compose-rabbitmq-cluster/)

    {{<gist yowko 95998a9f62af6dc2ff1b55c40013a798>}}

## 使用方式

1. 取得 container name

    - 如果使用範例的 docker compose，可以直接使用 `rabbitmq-1`、`rabbitmq-2`、`rabbitmq-3` 作為 container name

    - 如果未指定 container name 時 `docker ps` 可以看到目前正在執行的 container，這邊我們要取得 rabbitmq container 的 name

2. 設定 ansible inventory

    > 這邊的重點是 vars 要設定 `ansible_connection: docker`，這樣 ansible 才會知道要使用 docker 來連線

    {{< gist yowko 9374fd933e7ec71952c6182d67a71c26>}}

3. 設定 ansible playbook

    {{< gist yowko 490a0ad27825391663d394415097c573>}}

4. 執行 ansible playbook

    ```bash
    ansible-playbook -i test.ini test.yml
    ```

## 心得

實際效果：

![1readerpermission](https://github.com/yowko/picsbed/assets/3851540/c5819e8d-be7d-4b7b-8ddd-ae93f174fe0e)

![2writerpermission](https://github.com/yowko/picsbed/assets/3851540/29e3c77b-fc99-428d-bdc5-7555674784fd)

直接使用 container 來當作 ansible host 來執行 ansible playbook，節省非常大量的時間，並且可以確保每次的環境都是一致的，而且不需要額外的費用，重新建立環境也非常快速，這樣的方式可以讓我們更快速的開發 ansible playbook。

## 參考資料

1. [透過 docker compose 啟動 RabbitMQ cluster](/docker-compose-rabbitmq-cluster/)
2. [Specify docker containers in /etc/ansible/hosts file](https://stackoverflow.com/questions/47600633/specify-docker-containers-in-etc-ansible-hosts-file)
