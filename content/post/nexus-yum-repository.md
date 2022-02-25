---
title: "使用 Nexus 來建立 yum repository"
date: 2022-02-25T00:30:00+08:00
lastmod: 2022-02-25T00:30:31+08:00
draft: false
tags: ["Linux","CentOS"]
slug: "nexus-yum-repository"
---

## 使用 Nexus 來建立 yum repository

之前筆記 [使用 yum 下載包含相依套件的完整 package](/yum-download-package-with-dependency) 紀錄到如何將 package 與其相依 package 一次完整下載，但筆記最後也提到透過逐一安裝 package 實在太沒效率，所以今天就來紀錄一下如何使用 Nexus 自建 yum repository 以及實際的使用方式

## 基本環境說明

1. Azure VM Standard B2s (2 vcpu，4 GiB 記憶體)
2. Azure Linux image

    - cognosys/centos-7-9-free

3. docker-1.13.1-208.git7d71120.el7_9.x86_64
4. docker images

    - sonatype/nexus3:3.37.3

5. 使用 container image 啟動 nexus

    ```bash
    docker run -d -p 8081:8081 --name nexus sonatype/nexus3:3.37.3
    ```

## 建立與使用方式

1. 建立 yum repository

    ![1step1](https://user-images.githubusercontent.com/3851540/155702088-11680e27-bbc4-4fc5-9d60-f75d62efbb71.png)

    ![2step2](https://user-images.githubusercontent.com/3851540/155702107-52d5abc3-62a3-4bbb-b6cd-b2e9c2c3302c.png)

    > - 指定識別名稱 (加入 yum repo 時會用到)：這邊以 `yum` 為例
    > - 設定資料夾階層 (設定 rpm 在資料夾的階層數)：這邊以 `0` 為例，即 `http://<serveraddress:port>/repository/yum/{package name}.rpm`

2. 將 rpm 儲存至 nexus yum repository 中

    - 語法

        ```bash
        curl -v --user '{帳號}:{密碼}' --upload-file {package rpm 位置} http://{serveraddress}:{port}/repository/{前面設定的識別名稱}/{package}.rpm
        ```

    - 範例

        ```bash
        curl -v --user 'admin:pass.123' --upload-file ./jq-1.6-2.el7.x86_64.rpm http://localhost:8081/repository/yum/jq-1.6-2.el7.x86_64.rpm
        ```

3. 使用端加入 yum reposipory

    > 如果 nexus 設定不允許匿名存取，記得加入 repo 時要設定帳號密碼

    - 語法

        ```bash
        cat <<EOF > /etc/yum.repos.d/{repo name}.repo
        [{repo name}]
        name={repo name}
        baseurl=http://{serveraddress}:{port}/repository/{前面設定的識別名稱}/
        enabled=1
        gpgcheck=0
        [username={帳號}]
        [password={密碼}]
        EOF
        ```

    - 範例

        ```bash
        cat <<EOF > /etc/yum.repos.d/nexus.repo
        [nexus]
        name=nexus
        baseurl=http://192.168.1.2:8081/repository/yum/
        enabled=1
        gpgcheck=0
        username=admin
        password=pass.123
        EOF
        ```

## 心得

我不清楚 yum repository gpg check 的機制，我先把 `gpgcheck` 關掉了，也就沒給 `gpgkey` 先求個能用，待日後慢慢優化囉

## 參考資訊

1. [使用 yum 下載包含相依套件的完整 package](/yum-download-package-with-dependency)
2. [Nexus Repository Administration:Yum Repositories](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/yum-repositories)
