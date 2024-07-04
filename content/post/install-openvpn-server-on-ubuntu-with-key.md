---
title: "在 Ubuntu 上安裝有密碼保護的 OpenVPN Server"
date: 2024-06-05T00:30:00+08:00
lastmod: 2024-06-05T00:30:31+08:00
draft: false
tags: ["ubuntu","openvpn"]
slug: "install-openvpn-server-on-ubuntu-with-key"
---

## 在 Ubuntu 上安裝有密碼保護的 OpenVPN Server

在之前筆記 [在 Ubuntu 上安裝 OpenVPN Server](/install-openvpn-server-on-ubuntu) 紀錄到如何在 Ubuntu 上安裝 OpenVPN Server，但身為一個有資安意識的工程師，使用時總覺得怪怪的，整個過程就是這麼流暢，順到讓人覺得害怕呀

沒錯，我們沒有為 OpenVPN Server 設定任何保護機制，任何人拿到 client 的設定檔都可以透過我們建立的 OpenVPN Server 在網路上遨遊XD

為了避免 OpenVPN Server 的 cleint 設定檔被任意傳播造成不必要的麻煩，所以今天就來紀錄如何在 OpenVPN 連線時加上一個簡單的密碼驗證

## 基本環境說明

- Azure VM Standard B2s (2 vcpus, 4 GiB memory)
- canonical 0001-com-ubuntu-server-jammy 22_04-lts-gen2
- openvpn 2.5.9-0ubuntu0.22.04.2

## 安裝步驟

1. 取得 server 對外 IP

    > 用來做為 OpenVPN Server 的連線 ip，也用來為相關服務開通的來源 ip

    ```bash
    curl ifconfig.me
    ```

    ![1ip](https://github.com/yowko/picsbed/assets/3851540/0e918132-a9e6-4f2c-a09e-bdfc4e1d8e1a)

2. 使用安裝腳本進行安裝

    - 下載腳本

        ```bash
        wget https://raw.githubusercontent.com/angristan/openvpn-install/master/openvpn-install.sh -O openvpn-ubuntu-install.sh
        ```

        ![2wget](https://github.com/yowko/picsbed/assets/3851540/549b1a60-24c0-408f-9c16-a9265b69a25b)

    - 執行腳本

        ```bash
        sudo bash openvpn-ubuntu-install.sh
        ```

3. 逐步設定

    - 設定 OpenVPN Server 的連線 ip

        > 以我的例子，預設是內網 ip，所以需要手動輸入對外 ip

        ![3openvpnip](https://github.com/yowko/picsbed/assets/3851540/2652182c-69c4-4196-9c32-76db23794ae4)

    - 是否啟用 IPv6

        > 預設為 `n`，可以根據需求選擇 `y` 或 `n`

        ![4ipv6](https://github.com/yowko/picsbed/assets/3851540/f255efb3-ca91-4702-9ef9-66078c37ffcb)

    - 選擇連線 port

        > 根據需求設定 port，預設為 `1194`

        ![5port](https://github.com/yowko/picsbed/assets/3851540/537c8380-968a-486d-aa52-37272ae82502)

    - 選擇連線 protocol

        > 可以根據需求選擇 TCP 或 UDP，預設為 `UDP`

        ![6udp](https://github.com/yowko/picsbed/assets/3851540/9f942f30-e0c5-48f0-bab4-e30380032338)

    - 選擇 DNS

        > 根據需求選擇 DNS，預設為 `AdGuard DNS (Anycast: worldwide)`

        ![7dns](https://github.com/yowko/picsbed/assets/3851540/a6432a7d-0cb7-473c-9134-1881bd4f0a3b)

    - 是否啟用壓縮

        > 預設為 `n`，可以根據需求選擇 `y` 或 `n`

        ![8compress](https://github.com/yowko/picsbed/assets/3851540/eef04cab-7863-4d41-a1a2-bf4fd5f76cfb)

    - 是否啟用 custom encryption

        > 預設為 `n`，可以根據需求選擇 `y` 或 `n`

        ![9encrypt](https://github.com/yowko/picsbed/assets/3851540/09502529-a0ec-4726-9186-c40c1ff11840)

    - 執行安裝

        ![10install](https://github.com/yowko/picsbed/assets/3851540/4d7ed55b-37d7-4975-8b5d-89a05202fe59)

    - 設定 client name

        > 輸入 client name，`必填`

        ![11client](https://github.com/yowko/picsbed/assets/3851540/1a9e8cfa-cdc1-430f-bbd4-d7fb880d294c)

    - 是否啟用密碼保護 client 設定

        > 預設為 `1) Add a passwordless client`，可以根據需求選擇 `1) Add a passwordless client` 或 `2) Use a password for the client`

        ![12password](https://github.com/yowko/picsbed/assets/3851540/d1d1e757-b5f1-4045-8091-65ea7574fa28)

    - 設定密碼 (如果選擇 `Use a password for the client`)

        ![13pass](https://github.com/yowko/picsbed/assets/3851540/29b36e1a-c798-49d2-8066-bbfb067cf0cc)

    - client 設定完成

        > 設定完成後，會顯示相關訊息，包含 client 的設定檔位置

        ![14done](https://github.com/yowko/picsbed/assets/3851540/8815e0aa-d1ea-42dd-8dd5-eb6e18ac626e)

4. 匯入 client 設定檔

    - 下載 client 設定檔

    - 匯入 client 設定檔

        > 透過 OpenVPN Client 匯入 client 設定檔，並設定 Private Key Password

        ![15import](https://github.com/yowko/picsbed/assets/3851540/10f101ac-8c1b-49b2-ae03-b0fb8d49d407)

## 心得

- 實際效果：連結 vpn 後，對外 ip 會變成 OpenVPN Server 的 ip，這樣就可以透過 OpenVPN Server 來限制存取服務的來源 ip

    - 未使用 OpenVPN Server

        ![10before](https://github.com/yowko/picsbed/assets/3851540/966f9957-7366-49a0-81e3-9b7d0be91d2b)

    - 使用 OpenVPN Server

        ![11after](https://github.com/yowko/picsbed/assets/3851540/5e39193e-5c94-46cc-a404-891b7cd820e5)

- 新增移除 client 或是刪除 OpenVPN，可以透過重新執行腳本來操作

    ```bash
    sudo bash openvpn-ubuntu-install.sh
    ```

    ![16addrevoke](https://github.com/yowko/picsbed/assets/3851540/01cb6847-0c68-4ebe-85af-795c95a37a4e)

## 參考資料

1. [Ubuntu 20.04 LTS Set Up OpenVPN Server In 5 Minutes](https://www.cyberciti.biz/faq/ubuntu-20-04-lts-set-up-openvpn-server-in-5-minutes/#Install_OpenVPN_server)
2. [在 Ubuntu 上安裝 OpenVPN Server](/install-openvpn-server-on-ubuntu)
