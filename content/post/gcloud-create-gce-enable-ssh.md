---
title: "使用 gcloud shell 建立 GCE 並設定允許使用 ssh 連線"
date: 2020-11-22T21:30:00+08:00
lastmod: 2020-11-22T21:30:31+08:00
draft: false
tags: ["GCP"]
slug: "gcloud-create-gce-enable-ssh"
---

## 使用 gcloud shell 建立 GCE 並設定允許使用 ssh 連線

最近在評估 GCP 的可行性，雖然雲端上有許多可用的 SaaS，但原本都是自建服務，要花時間去了解如何設定與調整 SaaS (主要是成本面的考量)，所以在確認對 SaaS 有一定的掌握度前先透過 GCE 建立 vm 來架設服務，避免使用成本超乎預期

之前實體的 ansible 腳本也可以應用在 GCE 上，只是需要額外指定 ssh keyfile，為了避免遺忘，快速筆記一下

## 基本環境說明

1. macOS Catalina 10.15.7
2. gcloud SDK 319.0.0

## 設定方式

1. 設定 SSH keys

    - gen key

        > 這邊的 username 需符合 gcp account

        ```bash
        ssh-keygen -t rsa -f ~/.ssh/gcp-ssh-key -C {username}
        ```

    - get key

        ```bash
        cat ~/.ssh/gcp-ssh-key.pub
        ```

    - import to gcp

        ![importkey](https://user-images.githubusercontent.com/3851540/99950171-1a64a280-2db7-11eb-84f4-c05e79dc0929.png)

2. 建立 GCE instance

    - 語法

        ```bash
        gcloud compute instances create {vm name} --machine-type {machine type name} --image {image name} --image-project {project name}
        ```

        - 列出可用的 machine-type： `gcloud compute machine-types list`
        - 列出可用的 image 與對應的 image-project： `gcloud compute images list`

    - 範例

        ```bash
        gcloud compute instances create test1 --machine-type e2-standard-2 --image centos-8-v20201112 --image-project centos-cloud
        ```

3. 啟用 ssh

    - 語法

        ```bash
        gcloud compute instances add-metadata {vm name} --metadata serial-port-enable=TRUE --zone={zone name} --project={project_id}
        ```

        - 列出可用的 zone： `gcloud compute zones list`
        - 列出該帳號的 project： `gcloud projects list`

    - 範例

        ```bash
        gcloud compute instances add-metadata test1 --metadata serial-port-enable=TRUE --zone=asia-east2-a --project=testproject
        ```

- asnbile 指定 ssh key 的語法

    ```bash
    ansible-playbook --private-key /~/.ssh/gcp-ssh-key -i inventories/gcp.ini playbooks/test.yml -b
    ```

## 心得

設定還算直覺，文件也好找，整體使用體驗不錯，只是每個平台的使用邏輯不太相同，需要花點時間熟悉一下

我在建立 GCE instance 時，中間有個 step 可以建立設定 SSH，我嘗試統一由 gcloud 完成 GCE 建立並設定 SSH，但沒有成功，我初步看起來是 key 的格式跟名稱錯誤，但沒有仔細 debug，先列下來備忘

## 參考資訊

1. [gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create)
2. [Setting up OS Login](https://cloud.google.com/compute/docs/instances/managing-instance-access#gcloud)
