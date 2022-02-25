---
title: "使用 yum 下載包含相依套件的完整 package"
date: 2022-02-24T00:30:00+08:00
lastmod: 2022-02-24T00:30:31+08:00
draft: false
tags: ["Linux","CentOS"]
slug: "yum-download-package-with-dependency"
---

## 使用 yum 下載包含相依套件的完整 package

production 環境上的所有 server 因為資安問題沒有開放主動對外連線，連帶著 package 的安裝就沒辦法透過公用 yum repository 的方式來進行，所以興起自建 yum repository 的念頭

在實際建立 yum repository 之前需要先蒐集用到的 yum package，而現在許多套件都有其他套件的相依，單純只下載需要的套件本身是無法順利使用的，今天就來紀錄一下該如何下載完整的套件

## 基本環境說明

1. Azure VM Standard B2s (2 vcpu，4 GiB 記憶體)
2. Azure Linux image

    - cognosys/centos-7-9-free

3. yum packages

    - jq-1.6

## 使用方式

1. 透過 `deplist` 可以列出套件相依

    > 如果不想逐一搜尋所有 yum repository , 可以加上 `--enablerepo` 指定 repo

    - 語法

        ```bash
        yum deplist {package name} [--enablerepo={repo name}]
        ```

    - 範例

        ```bash
        yum deplist jq --enablerepo=epel
        ```

        ![1deplist](https://user-images.githubusercontent.com/3851540/155678353-e41b9538-7b6a-4912-9e25-15a5bc15c45e.png)

2. 完整下載指定套件與其相依套件

    - 語法

        - `--downloadonly` : 用來下載 package 而不安裝
        - `--installroot` : 需要指定一個空資料夾強迫 yum 認定所有相依皆不存在，否則只會下載系統不存在的相依套件
        - `--downloaddir` : 指定下載回來的套件存放路徑
        - `--releasever` : 指定 linux 版本，不加這個參數會出現存取 `$releasever` 但找不到的 404 的錯誤

            ```txt
            http://olcentgbl.trafficmanager.net/centos/%24releasever/os/x86_64/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
            Trying other mirror.
            To address this issue please refer to the below wiki article
            
            https://wiki.centos.org/yum-errors
            
            If above article doesn't help to resolve this issue please use https://bugs.centos.org/.
            
            http://olcentgbl.trafficmanager.net/centos/%24releasever/extras/x86_64/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
            Trying other mirror.
            http://olcentgbl.trafficmanager.net/centos/%24releasever/updates/x86_64/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
            Trying other mirror.
            http://olcentgbl.trafficmanager.net/openlogic/%24releasever/openlogic/x86_64/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
            ```

            ![2errno14](https://user-images.githubusercontent.com/3851540/155678366-e9678ca0-f166-43f7-8224-7425fd9dfce2.png)

        ```bash
        yum install --downloadonly --installroot={empty foleder path} --releasever=$(rpm --eval '%{centos_ver}') --downloaddir={rpm 儲放路徑} {package name}
        ```

    - 範例

        ```bash
        yum install --downloadonly --installroot=/tmp/jqrepo --releasever=$(rpm --eval '%{centos_ver}') --downloaddir=$(pwd) jq
        ```

    - 實際下載效果

        ![3downloadlist](https://user-images.githubusercontent.com/3851540/155678373-5df884d4-83c9-41dc-a0a8-728f3a4c808c.png)

## 心得

雖然最後的語法算是單純，但過程比想像中複雜：參數比較預期的多  眉眉角角也不少

在取得完整相依套件後，其實透過逐一安裝相依但件的方式最後就可以成功安裝目標套件，但這樣的方式不是很聰明也很沒效率，我還是比較偏好使用 yum repo 來統一管理

最後就是將下載回來的套件統統 import 進自建 yum repository 中就可以使用了

如果想自建 yum repository 可以參考 [使用 Nexus 來建立 yum repository](/nexus-yum-repository)

## 參考資訊

1. [yum 指定特定版本的Repository](https://tech.cv6.me/488/php/yum-%E5%AE%89%E8%A3%9D%E7%89%B9%E5%AE%9A%E7%89%88%E6%9C%AC%E7%9A%84repo.html)
2. [[Linux] 在 CentOS 上使用 yum，下載所有相關聯的 RPM 套件](https://ephrain.net/linux-%E5%9C%A8-centos-%E4%B8%8A%E4%BD%BF%E7%94%A8-yum%EF%BC%8C%E4%B8%8B%E8%BC%89%E6%89%80%E6%9C%89%E7%9B%B8%E9%97%9C%E8%81%AF%E7%9A%84-rpm-%E5%A5%97%E4%BB%B6/)
3. [使用 Nexus 來建立 yum repository](/nexus-yum-repository)
