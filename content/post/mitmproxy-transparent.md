---
title: "Mitmproxy 啟用 Transparent mode"
date: 2020-06-25T21:30:00+08:00
lastmod: 2020-06-25T21:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "mitmproxy-transparent"
---

## Mitmproxy 啟用 Transparent mode

之前筆記 [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy) 提到如果每次 request 都需要指定 proxy，會讓 proxy 在使用上的便利性大打折扣，所以可以將 proxy 設為 trasparnet mode 並搭配 nat 設定從網路層攔劫特定 request，今天就來紀錄一下 Mitmproxy 如何設定 trasparnet mode

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. Mitmproxy v5.1.1
4. Python 3.6.8
5. OpenSSL 1.1.1g  21 Apr 2020

## 設定方式

> 我使用單一 server 來模擬，設定上略有不同

1. 確保 os 允許 IP Forward

    > 如果希望重開機仍保留設定，需要修改 `/etc/sysctl.conf`

    ```bash
    sysctl -w net.ipv4.ip_forward=1
    sysctl -w net.ipv6.conf.all.forwarding=1
    ```

2. 建立啟動 mitmproxy 用 user

    > 沒有建立專用 user，在 iptable 設定時就無法針對 mitmproxy 的 request 做排除，會出現無窮迴圈

    ```bash
    sudo useradd --create-home mitmproxyuser
    sudo -u mitmproxyuser bash -c 'cd ~ && pip3 install --user mitmproxy'
    ```

3. iptable

    ```bash
    iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 80 -j REDIRECT --to-port 8080
    iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 443 -j REDIRECT --to-port 8080
    ip6tables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 80 -j REDIRECT --to-port 8080
    ip6tables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner mitmproxyuser --dport 443 -j REDIRECT --to-port 8080
    ```

4. 調整啟動方式

    > `Mitmproxy` 要設定成 trasparnet mode 的方式很簡單，只要在啟動 Mitmproxy 時加上 `--mode transparent`，不過因為 iptable nat 這層需要額外動手腳，所以會多些步驟

    ```bash
    sudo -u mitmproxyuser bash -c '$HOME/.local/bin/mitmproxy --mode transparent'
    ```

5. https

    > transparent mode 下的 https 讓我吃了點苦頭：一直遇到憑證無法通過 curl 驗證問題

    - 複製憑證

        > 將之前筆記 [Mitmproxy 啟用 Https](https://blog.yowko.com/mitmproxy-https) 建立的憑證複製新建立的 mitmproxy 執行 user 預設目錄中

        ```bash
        /bin/cp ~/.mitmproxy/*.* /home/mitmproxyuser/.mitmproxy/
        ```

    - 重新啟動 mitmproxy

        ```bash
        sudo -u mitmproxyuser bash -c '$HOME/.local/bin/mitmproxy --mode transparent'
        ```

## 心得

不確定是不是我設定的情境比較特別，找了些文章都沒有解決我的問題：`curl` https resource 會出現憑證驗證失敗，雖然在執行 curl 時加上 `-k` 或是 `--insecure` 就可以暫時避開問題，但就是覺得不夠漂亮，最後才試出複製憑證的方法，不過我還是沒有很清楚整個機制，大意上跟 [Mitmproxy 啟用 Https](https://blog.yowko.com/mitmproxy-https) 的流程相同，而不同 user 會讀取不同的憑證路徑，所以將建立過的憑證複製至新 user 的憑證位置就可以了

但最後還是沒有選用 mitmproxy，原因是 mitmproxy 不支援同時使用 upstream 跟 transparent

## 參考資訊

1. [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy)
2. [Mitmproxy 啟用 Https](https://blog.yowko.com/mitmproxy-https)
3. [Transparent Proxying](https://docs.mitmproxy.org/stable/howto-transparent/)
