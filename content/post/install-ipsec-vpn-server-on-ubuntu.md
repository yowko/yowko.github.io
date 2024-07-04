---
title: "在 Ubuntu 上安裝 IPsec VPN Server"
date: 2024-06-21T00:30:00+08:00
lastmod: 2024-06-21T00:30:31+08:00
draft: false
tags: ["ubuntu","openvpn"]
slug: "install-ipsec-vpn-server-on-ubuntu"
---

## 在 Ubuntu 上安裝 IPsec VPN Server

之前筆記 [在 Ubuntu 上安裝 OpenVPN Server](/install-openvpn-server-on-ubuntu) 與 [在 Ubuntu 上安裝有密碼保護的 OpenVPN Server](/install-openvpn-server-on-ubuntu-with-key) 紀錄在如何在 Ubuntu 上安裝 OpenVPN Server，基本上已經可以滿足透過特定 ip 存取服務的需求，只是在某些情境我們會遇到同時要透過兩個 ip 存取不同服務，所以希望可以讓兩個 vpn 同時啟用，順帶也學習一下不同 VPN server 的建立方式做為技術儲備，今天就來紀錄如何在 Ubuntu 上安裝 IPsec VPN Server。

我看 [GitHub - hwdsl2/setup-ipsec-vpn](https://github.com/hwdsl2/setup-ipsec-vpn) 提到支援 IPsec/L2TP 與 IKEv2，但我不太清楚這兩者之間的差異，所以用 chatgpt 查了一下，以下是它們的差異：

IKEv2 和 L2TP/IPsec 是兩種常用的 VPN 協議，這兩者在功能、性能和安全性上各有優劣。以下是它們的主要差異和優缺點：

- IKEv2 (Internet Key Exchange version 2)

    IKEv2 是一種 VPN 協議，通常與 IPsec 一起使用，專門用於提供安全的鍵交換和隧道管理。

    - 優點：
        1. 速度和效能：

            - IKEv2 對於移動設備的支持非常出色，因為它具有 MOBIKE（Mobility and Multihoming）功能，允許 VPN 在網絡變更時自動重新連接（如從 Wi-Fi 切換到移動數據）。
            - 通過效率更高的加密和鍵交換機制，它的連接速度和效能通常比 L2TP/IPsec 更好。
        2. 安全性：

            - 提供強大的安全功能，包括 AES 加密和 SHA-2 驗證。
            - 支持多種身份驗證方式，包括數字證書和預共享密鑰。
        3. 穩定性和可靠性：

            - 在不穩定的網絡環境下也能保持穩定連接，特別適合移動用戶。
    - 缺點：
        1. 兼容性：

            - 雖然大多數現代操作系統都支持 IKEv2，但某些較舊的設備和系統可能不支持。
        2. 設置複雜性：

            - 對於一些初學者來說，設置和配置 IKEv2 可能會比較複雜。
- L2TP/IPsec (Layer 2 Tunneling Protocol over IPsec)
    L2TP 是一種隧道協議，通常與 IPsec 結合使用，以提供加密和安全性。

    - 優點：
        1. 兼容性：

            - 廣泛支持於多種操作系統和設備上，包括較舊的系統和設備。
            - 通常內建於 Windows、macOS、Linux 和多數移動操作系統中，設置較為簡單。
        2. 安全性：

            - 結合 IPsec 提供雙層保護：L2TP 用於隧道化，IPsec 用於加密。
            - 提供較高的安全性，適用於大多數企業環境。
    - 缺點：
        1. 速度和效能：

            - 相比 IKEv2，L2TP/IPsec 的連接建立過程較慢，因為涉及兩層封裝和加密。
            - 由於雙層協議的開銷，效能上通常不如 IKEv2。
        2. NAT 穿透問題：

            - 在某些 NAT 環境中，L2TP/IPsec 可能會遇到連接問題，需要額外的配置來解決。
- 總結
    1. IKEv2：適合需要高效能和穩定連接的用戶，特別是移動設備用戶。雖然設置可能稍微複雜，但提供了更好的速度和安全性。
    2. L2TP/IPsec：適合需要廣泛兼容性和易於設置的用戶，尤其是在需要支持多種舊設備的情況下。然而，效能和速度上略遜於 IKEv2。

根據具體需求和環境選擇合適的 VPN 協議，可以在安全性、速度和兼容性之間取得平衡。

## 基本環境說明

- Azure VM Standard B2s (2 vcpus, 4 GiB memory)
- canonical 0001-com-ubuntu-server-jammy 22_04-lts-gen2
- openvpn 2.5.9-0ubuntu0.22.04.2
- macOS Sonoma 14.5 (Apple M2 Pro)

## 設定方式

1. 安裝 IPsec VPN Server (預設是 `IKEv2`)

    預設安裝即會產生一組 client： `vpnclient`

    ```bash
    wget https://get.vpnsetup.net -O vpn.sh && sudo sh vpn.sh
    ```

    ![1install](https://github.com/yowko/picsbed/assets/3851540/ba4827c4-fef2-4441-8a46-d2c6e3a5f185)

2. 設定防火牆 allow 500/udp 與 4500/udp

3. macOS 安裝 client

    直接下載 `vpnclient.mobileconfig` 並雙擊開啟，再至 `隱私權與安全` --> `描述檔` 安裝

    ![5payloadinstall](https://github.com/yowko/picsbed/assets/3851540/1a6558f5-b910-4537-953d-3b8132873626)

## 心得

1. 新增 client

    ```bash
    sudo ikev2.sh
    ```

    ![2addclient](https://github.com/yowko/picsbed/assets/3851540/275799c9-3270-4b72-9867-9b21141215f8)

2. 調整顯示名稱

    直接修改 `.mobileconfig` 檔案 (xml 格式)，以上述的 `vpnclient` 為例為 `vpnclient.mobileconfig`

    - 描述檔顯示名稱：plist -> dict -> key -> PayloadDisplayName -> string

        ![3payloadfile](https://github.com/yowko/picsbed/assets/3851540/02d9b15f-fdd4-4270-a7bc-9ebdd8edc90d)

    - VPN 顯示名稱： plist -> dict -> array -> dict- > key -> UserDefinedName -> string

        ![4vpnname](https://github.com/yowko/picsbed/assets/3851540/423ff948-6c46-484f-9e61-3a7d2b28c2a1)

## 參考資訊

1. [在 Ubuntu 上安裝 OpenVPN Server](/install-openvpn-server-on-ubuntu)
2. [在 Ubuntu 上安裝有密碼保護的 OpenVPN Server](/install-openvpn-server-on-ubuntu-with-key)
3. [GitHub - hwdsl2/setup-ipsec-vpn](https://github.com/hwdsl2/setup-ipsec-vpn)
