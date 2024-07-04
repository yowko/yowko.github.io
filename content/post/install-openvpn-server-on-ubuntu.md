---
title: "在 Ubuntu 上安裝 OpenVPN Server"
date: 2024-06-04T00:30:00+08:00
lastmod: 2024-06-04T00:30:31+08:00
draft: false
tags: ["ubuntu","openvpn"]
slug: "install-openvpn-server-on-ubuntu"
---

## 在 Ubuntu 上安裝 OpenVPN Servern

最近產品有些服務的異常存取增加，根據網站收到的 request log 數量與參數來看，推測是有人在嘗試找出系統上的 api 漏洞或是破解帳密，因此決定將服務的存取限制在特定 IP 之下，而這個需求可以透過 OpenVPN 來達成。

今天就來快速紀錄一下在 Ubuntu 上安裝 OpenVPN Server 的步驟

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
        wget https://git.io/vpn -O openvpn-ubuntu-install.sh
        ```

        ![2wget](https://github.com/yowko/picsbed/assets/3851540/0d5bcaea-59b3-4402-9c49-9130d5a9046e)

    - 執行腳本

        ```bash
        sudo bash openvpn-ubuntu-install.sh
        ```

3. 逐步設定

    - 設定 OpenVPN Server 的連線 ip

        > 如果偵測到的 ip 不是預期的 ip，可以手動輸入進行調整

        ![3openvpnip](https://github.com/yowko/picsbed/assets/3851540/1eb2d804-6e25-4f0b-981d-643fbd9a2e8d)

    - 選擇連線 protocol

        > 可以根據需求選擇 TCP 或 UDP，預設為 `UDP`

        ![4udp](https://github.com/yowko/picsbed/assets/3851540/ed3c8275-b3c2-430e-ac26-89ad49ba03c2)

    - 選擇連線 port

        > 根據需求設定 port，預設為 `1194`

        ![5port](https://github.com/yowko/picsbed/assets/3851540/5302a72c-9db6-4b87-84d9-bd55f5e75da1)

    - 選擇 DNS

        > 根據需求選擇 DNS，預設為 `Current system resolvers`

        ![6dns](https://github.com/yowko/picsbed/assets/3851540/129cd395-4321-47a5-9d9a-b1ffbd6702ff)

    - 設定 client name

        > 輸入 client name，預設為 `client`，這會是 client 的檔名

        ![7client](https://github.com/yowko/picsbed/assets/3851540/13d17e1d-afee-4ea9-9dec-7088ebed80f0)

    - 完成安裝

        > 安裝完成後，會顯示相關訊息，包含 client 的設定檔位置

        ![8done1](https://github.com/yowko/picsbed/assets/3851540/cad7a901-1a00-4d22-b229-015cfd679bca)

        ![8done2](https://github.com/yowko/picsbed/assets/3851540/59ecddde-d2bb-4037-a1f9-8e75bbd6e269)

4. 匯入 client 設定檔

    - 下載 client 設定檔

    - 匯入 client 設定檔

        > 透過 OpenVPN Client 匯入 client 設定檔

        ![9import](https://github.com/yowko/picsbed/assets/3851540/f45b57ed-04a1-466a-a776-3a86f643b95f)

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

    ![12addrevoke](https://github.com/yowko/picsbed/assets/3851540/05083eea-bcd9-4674-837a-c025ef885b6b)

## 參考資料

1. [Ubuntu 20.04 LTS Set Up OpenVPN Server In 5 Minutes](https://www.cyberciti.biz/faq/ubuntu-20-04-lts-set-up-openvpn-server-in-5-minutes/#Install_OpenVPN_server)
