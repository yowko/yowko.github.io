---
title: "Squid Proxy Https 設定"
date: 2020-06-14T21:30:00+08:00
lastmod: 2020-06-14T21:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "squid-proxy-https"
---

## Squid Proxy Https 設定

依據之前筆記 [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy) 比照相同方式設定 https 的 request 無法取得正確回應

> 以下設定方式直接套用，會出現錯誤

```bash
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -m owner --uid-owner squid -j RETURN
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -j REDIRECT --to-ports 3129
```

測試了幾個做法，發現很容易查到錯誤設定方式，立馬自行紀錄一下

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. Squid 3.5.20

## 錯誤訊息

> 無法直接存取 https 的頁面

1. 訊息內容

    ```txt
    curl: (35) SSL received a record that exceeded the maximum permissible length.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/84597911-9ddda780-ae99-11ea-96c5-698de2c8b3a6.jpg)

## 設定方式

1. 建立憑證

    ```bash
    mkdir -p /etc/squid/cert/
    cd /etc/squid/cert/
    openssl req -new -newkey rsa:4096 -sha256 -days 365 -nodes -x509 -keyout myCA.pem -out myCA.pem
    openssl x509 -in myCA.pem -outform DER -out myCA.der
    ```

2. 初始化憑證資料夾

    ```bash
    /usr/lib64/squid/ssl_crtd -c -s /var/lib/ssl_db
    chown -R squid /var/lib/ssl_db
    ```

    > 沒有初始化的錯誤

    - 錯誤訊息

        ```bash
        The ssl_crtd helpers are crashing too rapidly, need help!
        ```

    - 錯誤截圖

        >![2noiniterror](https://user-images.githubusercontent.com/3851540/84597912-9f0ed480-ae99-11ea-8d1f-32338b56633e.jpg)

3. 修改 config

    ```bash
    vi /etc/squid/squid.conf
    ```

    > 加上以下設定

    ```txt
    acl step1 at_step SslBump1
    acl step2 at_step SslBump2
    acl step3 at_step SslBump3
    ssl_bump stare step2
    ssl_bump bump step3
    sslproxy_cert_error allow all
    https_port 3130 intercept ssl-bump  generate-host-certificates=on dynamic_cert_mem_cache_size=4MB cert=/etc/squid/cert/myCA.pem key=/etc/squid/cert/myCA.pem
    sslcrtd_program /usr/lib64/squid/ssl_crtd -s /var/lib/ssl_db -M 4MB
    ```

4. iptable 設定

    ```bash
    iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -m owner --uid-owner squid -j RETURN
    iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -j REDIRECT --to-ports 3130
    ```

5. 設定成功

    ![3success](https://user-images.githubusercontent.com/3851540/84597914-a0400180-ae99-11ea-9046-41aa357f4147.jpg)

## 心得

squid 的 https 設定難度高上許多，主要是憑證這段我不懂，東卡西卡的，遇到錯誤也不知道怎麼解決，所幸勤能補拙，總算試出個方式，雖然不了解其中涵意，就待日後機緣到了，相信會懂的XD

另個問題是 squid 不支援 https 與 http 的互相轉換，在使用上還是有限制 (內網都只有 http 但外部連結都只提供 https)，透過 nat 的 tranparent proxy 攔 https request 就只能用 https 轉發，這個問題就留到下篇筆記囉

## 參考資訊

1. [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy)
2. [SQUID FATAL: The ssl_crtd helpers are crashing too rapidly, need help!](https://www.linux.org.ru/forum/admin/13819277)
3. [ssl_crtd helpers are crashing too rapidly in squid](https://serverfault.com/questions/624879/ssl-crtd-helpers-are-crashing-too-rapidly-in-squid)
4. [Squid proxy - a short guide (forward & transparent proxy examples, SSL bumping, links to guides)](https://www.reddit.com/r/sysadmin/comments/a67hly/squid_proxy_a_short_guide_forward_transparent/)
