---
title: "設定 Azure Portal 存取 Azure Storage Files"
date: 2019-09-14T01:30:00+08:00
lastmod: 2019-09-14T01:30:31+08:00
draft: false
tags: ["Azure"]
slug: "azure-portal-access-azure-files"
---

## 設定 Azure Portal 存取 Azure Storage Files

一直以來我需要 Linux 環境時都是建立在 Azure 上，一來是沒有自己的硬體，二來是可以無痛快速重建整個環境，練習 Kubernetes 更是如此，可以滿足動態增減機器的需求，缺點就是費用會隨著使用時間增加。

最近因為專案需要使用到 Kubernetes - Persistent Volume，過去只用過 hostpath，在嘗試 Azure Files 時遇到一個 Azure 的設定問題，所以額外筆記一下

## 重現問題：Access denied

1. 在 Azure 上建立 Storage Account

    ![1storageaccount](https://user-images.githubusercontent.com/3851540/64909833-883d2500-d743-11e9-9614-49e5f5f2822c.png)

    > 設定請依實際情境調整，以下僅提供個人使用範例

    ![2setting](https://user-images.githubusercontent.com/3851540/64909834-883d2500-d743-11e9-878a-ef7f38a7d32c.png)

2. Storage accpunts --> 選擇目標 account --> Files

    - 錯誤訊息

        ```txt
        Access denied

        You do not have access

        Server failed authenticate the request. Make sure the value of Authorization header is formed correctly including the signature.
        ```

    - 錯誤截圖

        ![3accessdeny](https://user-images.githubusercontent.com/3851540/64909835-883d2500-d743-11e9-9a06-7c33339a554e.png)

3. Access denied 解決方式

   - Storage account --> 目標 account --> Access control (IAM)

        ![4iam](https://user-images.githubusercontent.com/3851540/64909836-883d2500-d743-11e9-8ff2-92f588f60317.png)

   - 設定權限

        > 可以接受的權限有
        > - Azure Resource Manager Owner
        > - Azure Resource Manager Contributor
        > - Storage Account Contributor

        ![6addrole](https://user-images.githubusercontent.com/3851540/64909838-88d5bb80-d743-11e9-80a1-ff97495adebb.png)

        ![5addroleassignment](https://user-images.githubusercontent.com/3851540/64909980-cab33180-d744-11e9-937f-f4a2fb23d280.png)

## 重現問題：Authorization failure

> Storage accpunts --> 選擇目標 account --> Files

- 錯誤訊息

    ```txt
    Authorization failure

    You do not have access

    An authorization failure occurred. Recent changes to 'Firewalls and virtual networks' settings may not be in effect yet. If you have recently changed these settings, try waiting for up to a minute, then revisit this experience. The error was: 'code: AuthorizationFailure.
    ```

- 錯誤截圖

    ![7thfail](https://user-images.githubusercontent.com/3851540/64909839-88d5bb80-d743-11e9-8f4a-a859205adaa8.png)

- Authorization failure 解決方式

    > 將目前使用 ip 加入 access 清單中： Firewalls and virtual networks --> Add your IP address

    ![8addclientip](https://user-images.githubusercontent.com/3851540/64909840-88d5bb80-d743-11e9-8bbc-ccc89038b839.png)

## 心得

每次需要用到雲端服務時難免會有些擔憂，不論是 Azure 還是 GCP 都一樣：我需要一個功能或是服務，看著畫面憑著直覺設定，卻不時遇到無法使用的狀況，有時是權限沒設定、有時是網路沒開通，雖然都是些小設定，但常常因為這些小設定卡了幾個小時，原因是訊息常常不夠明確，而官網文件也常跟畫面對不起來也是問題，除此之外 Azure 文件則是太偏介紹，文件裡連結連來連去，常常得花不少時間才能得到正確答案

## 參考資訊

1. [Use the Azure portal to access blob or queue data](https://docs.microsoft.com/en-us/azure/storage/common/storage-access-blobs-queues-portal)
2. [Azure Storage redundancy](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy)
3. [Azure storage account overview](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview)
4. [Create a storage account](https://docs.microsoft.com/en-us/azure/storage/common/storage-quickstart-create-account)
5. [Configure Azure Storage firewalls and virtual networks](https://docs.microsoft.com/en-us/azure/storage/common/storage-network-security)
