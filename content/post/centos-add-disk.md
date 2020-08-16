---
title: "在 Centos 上新增磁區"
date: 2020-08-08T20:30:00+08:00
lastmod: 2020-08-08T20:30:31+08:00
draft: false
tags: ["Azure","Linux"]
slug: "centos-add-disk"
---

## 在 Centos 上新增磁區

最近在調整公司的 Ansible 各 service 的安裝腳本，而身為工程師出身的 SRE，當然不能沒有測試呀。之前寫單一目的 ansible role 時，都是透過建立 azure vm 來進行測試，這次調整所有 service 也不例外，不過卻因為 service 太多，造成 azure vm 的內建 disk 爆滿，重建了幾次 vm 後，決定簡單紀錄一下，加速自己的操作

## 基本環境說明

1. Azure VM

    - Standard_B2S 2vcpu 4GiB memory

2. OS image
    - CentOS 7.7 Free (Cognosys Inc.)

3. 預設僅有 30g OS 暫存磁碟，加掛 128g 資料磁碟

    ![1adddisk](https://user-images.githubusercontent.com/3851540/89710747-43078e80-d9b8-11ea-9627-c2de7204132d.png)

## 設定方式

1. 確認磁區掛載狀況

    > 額外的 128g 資料磁碟尚未掛載

    ```bash
    df -lh
    ```

    ![2dflh](https://user-images.githubusercontent.com/3851540/89710748-46027f00-d9b8-11ea-86d2-a4b88a0a2dc5.png)

2. 確認當下 disk 狀態

    > `/dev/sdc` 還沒有分割

    ```bash
    fdisk -l
    ```

    ![3fdisk](https://user-images.githubusercontent.com/3851540/89710749-469b1580-d9b8-11ea-802e-95ec0f8c0800.png)

3. 分割磁碟

    > `/dev/sdc` 請依實際情況調整(可能不是在 `/dev/sdc`)

    ```bash
    fdisk /dev/sdc
    ```

    - `n`:新增分割區
    - `p`:`primary partition`
    - 直接 enter :default `1`
    - 直接 enter :start sector default 2048
    - 直接 enter :end sector default 最大值
    - `w`:重新讀取設定

    ![4fdisknew](https://user-images.githubusercontent.com/3851540/89710751-4733ac00-d9b8-11ea-9486-76341eb0262f.png)

4. 確認磁碟分割效果

    > 檢查磁碟分割是否成功

    ```bash
     fdisk -l /dev/sdc
    ```

    ![5fdiskcheck](https://user-images.githubusercontent.com/3851540/89710752-47cc4280-d9b8-11ea-840a-c813f2db4cdc.png)

5. format disk

    > 磁區名稱請依實際狀況調整

    ```bash
    mkfs -t xfs /dev/sdc1
    ```

    或是

    ```bash
    mkfs.xfs -f /dev/sdc1
    ```

    ![6format](https://user-images.githubusercontent.com/3851540/89710753-4864d900-d9b8-11ea-9de3-801e762d0b38.png)

6. 掛載

    > 將剛 format 完成的磁區掛載供使用

    - 取得磁區的 UUID

        ```bash
        blkid
        ```

        ![7blkid](https://user-images.githubusercontent.com/3851540/89710754-4864d900-d9b8-11ea-9211-be8fb924f5aa.png)

    - 修改 `/etc/fstab`

        > 將新增的磁區 UUID 掛載至指定的路徑下

        ```bash
        vi /etc/fstab
        ```

        ![8mount](https://user-images.githubusercontent.com/3851540/89710755-48fd6f80-d9b8-11ea-810c-aa1514384728.png)

7. 實際效果

    > 掛載後執行 `reboot` 或是 `mount /root` 就可以看到效果了

    ![9result](https://user-images.githubusercontent.com/3851540/89710756-48fd6f80-d9b8-11ea-9d5e-9a5d04d2316a.png)

## 心得

原本想要直接擴充 `/`，但一直試不出來，急著要測試腳本，就留待改天再試吧

查資料的過程中，一直找到 LVM (Logical Volume Manager)、VG ( Logical Volume Group)、PV (Physical Volume) 相關設從，但 azure vm 預設沒用到這些設定，派不上用場，不過就資料看起來比較接近原本我想要的方式

另外我就不懂 Azure 為什麼不開放我直接擴充 OS disk，讓我花了不少時間

## 參考資訊

1. [替 Linux 新增硬碟（磁碟分割、格式化與掛載）](https://blog.gtwang.org/linux/linux-add-format-mount-harddisk/)
