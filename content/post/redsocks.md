---
title: "安裝 Redsocks"
date: 2020-06-26T21:30:00+08:00
lastmod: 2020-06-26T21:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "redsocks"
---

## 安裝 Redsocks

之前一連串幾篇筆記分別紀錄到 `Squid` 與 `Mitmproxy` 的使用方式，但因為 `Squid` 不支援 https 轉 http、`Mitmproxy` 不支援 transparent 與 upstream 並行，最後找到 `Redsocks` 可以滿足使用情境，所以今天來紀錄 Redsocks 的設定

今天紀錄的部份是假設上層有 squid proxy 的情境下做的設定，請特別留意

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. libevent-devel 2.0.21
4. gcc 4.8.5

## 設定步驟

1. 建立執行 redsocks 專用 user

    > 這是為了後續方便控制 iptable，避免出現無窮迴圈的轉導

    ```bash
    useradd --create-home redsocksuser
    ```

2. 安裝相依套件

    ```bash
    yum install libevent-devel git gcc
    ```

3. 下載 Redsocks 原始碼

    > 我沒找到 rpm 可以直接安裝，需要自行 build source code

    ```bash
    git clone https://github.com/darkk/redsocks
    ```

4. 編譯 Redsocks

    ```bash
    cd ~/redsocks && make
    ```

5. 將編譯好的 redsocks binary 指定 user 可存取位置  

    ```bash
    cp redsocks /home/redsocksuser/
    ```

6. 在指定 user 可存取位置中建立 config

    ```bash
    vi /home/redsocksuser/redsocks.conf
    ```

    > 如需參考完整 confif 範例可以參考 `redsocks.conf.example`

    ```conf
    base {
        log_debug = on;
        log_info = on;
        log = stderr;
        daemon = off;
        redirector = iptables;
    }
    redsocks {

        local_ip = 127.0.0.1;
        local_port = 12345;

        // 以下是上層 proxy 的相關設定，請依實際情況修改
        ip = 10.0.0.12;
        port = 3128;
        type = http-connect;
        login = "yowko";
        password = "pass.123";
    }
    ```

7. 修改 iptables

    - 建立專屬的 chain

        ```bash
        iptables -t nat -N REDSOCKS
        ```

    - 將 user `root` (依個別情境調整) 發動對外的 port 為 `443` 的 request 導向 REDSOCKS

        ```bash
        iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner root --dport 443 -j REDSOCKS
        ```

    - 將 REDSOCKS 的 request 導向 port `12345`

        ```bash
        iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 12345
        ```

    - 如果想要開機直接套用

        > 儲存目前 iptable 設定

        ```bash
        iptables-save > /etc/sysconfig/iptables
        ```

        > 開機直接套用已儲存的 iptable 設定

        ```bash
         echo "iptables-restore < /etc/sysconfig/iptables" >> /etc/rc.d/rc.local && chmod +x /etc/rc.d/rc.local
        ```

8. 啟動 redsocks

    ```bash
    sudo -u redsocksuser bash -c '/home/redsocksuser/redsocks -c /home/redsocksuser/redsocks.conf'
    ```

## 心得

一連測試了 `Squid`、`Mtimproxy` 與 `Redsocks`，最後終於選定了使用 `Redsocks`，主因就是如之前說的 `Squid` 不支援 https 轉 http (原有系統上游已經建立 `Squid`，避免影響現有服務不想動它)、`Mitmproxy` 不支援 transparent 與 upstream 並行 (沒辦法只用 `Mitmproxy` 就將 request 導向上游的 `Squid`)

雖然最後成功達成目的，但過程比較像是運氣好，因為 `Redsocks` 比較知名的是使用 `Socks5`，而我一直試不出正確設定，試試 `http-connect` 還真的可行，至於為什麼 `Socks5` 為什麼不能用，我個人猜測是因為上游的 `Squid` 是 http proxy 的關係，但我沒把握，身為一個商管科系出身的工程師，網路相關知識都是我很難跨越的痛呀(仔細想想好像不止網路 ~~~)

## 參考資訊

1. [How to install and configure Redsocks on Centos Linux](https://crosp.net/blog/administration/install-configure-redsocks-proxy-centos-linux/)
2. [Centos 7 下配置 redsocks 代理](https://zhuanlan.zhihu.com/p/80942720)
3. [redsocks/redsocks.service](https://github.com/darkk/redsocks/blob/master/redsocks.service)
