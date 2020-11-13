---
title: "MAC 封鎖存取特定 IP"
date: 2020-11-13T21:30:00+08:00
lastmod: 2020-11-13T21:30:31+08:00
draft: false
tags: ["macOS","Network"]
slug: "mac-block-outgoing-ip"
---

## MAC 封鎖存取特定 IP

這個需求源自連不到某個共用服務時的 exception handle 測試，一般來說直接將 mac 斷網就可以模擬，但想要測試的服務被包在一段流程中，斷網會讓該流程一開始就 fail 沒辦法驗證這次的需求，因此在沒辦法直接斷開，又想測試某個服務無法使用的情境，造就了這個需求：在 mac 上封鎖針對某個 ip 的所有 request

## 基本環境說明

1. macOS Catalina 10.15.7

## 設定方式

1. 修改 PF firewall 設定檔

    ```bash
    sudo vi /etc/pf.conf
    ```

2. 加上想要 block 的 ip 規則

    - 語法

        ```txt
        block drop out quick proto {tcp or udp} from {source} to {target ip} [port {port}]
        ```

    - 範例

        1. 封鎖透過 tcp 存取特定 ip 所有 port

            ```txt
            block drop out quick proto tcp from any to 192.168.1.109
            ```

        2. 封鎖透過 tcp 存取特定 ip 特定 port

            ```txt
            block drop out quick proto tcp from any to 192.168.1.109 port 8500
            ```

3. 重新載入修改後的 PF firewall 設定

    ```bash
    sudo pfctl -f /etc/pf.conf
    ```

4. 啟用修改的 PF firewall 設定

    ```bash
    sudo pfctl -e
    ```

   - 如果需要停用 PF firewall

       ```bash
       sudo pfctl -d
       ```

5. 實際效果

    - 設定前：正常存取

        ![1before](https://user-images.githubusercontent.com/3851540/99077714-22268900-25f8-11eb-847b-02e52041a840.png)

    - 設定後：無法存取

        ![2after](https://user-images.githubusercontent.com/3851540/99077721-2488e300-25f8-11eb-9ba6-efa1291a003b.png)

## 心得

現在回頭看流程好簡單喔XD 只是看到這個需求時還是愣了一下，雖然 pf 不是第一次碰，但 macOS 的設定相對於 Linux 還是有些差異，遇過幾次不能直接套用，給我留下了陰影  哈

## 參考資訊

1. [How do I drop outgoing packets to specific host/port?](https://apple.stackexchange.com/a/230556)
