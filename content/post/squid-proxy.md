---
title: "安裝 Squid Proxy"
date: 2020-06-13T21:30:00+08:00
lastmod: 2020-06-13T21:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "squid-proxy"
---

## 安裝 Squid Proxy

專案有個需求是跟 partner 之間的溝通只限定於註冊過的 server ip，雖然在資安的考量下是很常見的做法，但對於實際的營運上不免會有些限制：像是如果只綁定某一台 server，這樣一來在服務快速成長的期間 server 擴展的即時性會比較差；如果只綁在 load balancer 上也有類似問題，沒辦法跨機房，這時候可以透過架設 proxy server 並將該 proxy server ip 註冊至 partner 端的方式來處理，這麼一來只要 proxy server 正常運作，後面 server 就可以擁有配置的彈性

過去都是 infra 同事協助設定，這次自己來，剛好趁這個機會學習一下

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. Squid 3.5.20

## 基本安裝

1. 透過 yum 安裝

    ```bash
    yum install -y squid
    ```

2. 調整 config

    > 預設位置 `/etc/squid/squid.conf`

    ```bash
    vi /etc/squid/squid.conf
    ```

    > 預設 config 如下

    ```conf
    #
    # Recommended minimum configuration:
    #

    # Example rule allowing access from your local networks.
    # Adapt to list your (internal) IP networks from where browsing
    # should be allowed
    acl localnet src 10.0.0.0/8	# RFC1918 possible internal network
    acl localnet src 172.16.0.0/12	# RFC1918 possible internal     network
    acl localnet src 192.168.0.0/16	# RFC1918 possible internal     network
    acl localnet src fc00::/7       # RFC 4193 local private    network range
    acl localnet src fe80::/10      # RFC 4291 link-local (directly     plugged) machines

    acl SSL_ports port 443
    acl Safe_ports port 80		# http
    acl Safe_ports port 21		# ftp
    acl Safe_ports port 443		# https
    acl Safe_ports port 70		# gopher
    acl Safe_ports port 210		# wais
    acl Safe_ports port 1025-65535	# unregistered ports
    acl Safe_ports port 280		# http-mgmt
    acl Safe_ports port 488		# gss-http
    acl Safe_ports port 591		# filemaker
    acl Safe_ports port 777		# multiling http
    acl CONNECT method CONNECT

    #
    # Recommended minimum Access Permission configuration:
    #
    # Deny requests to certain unsafe ports
    http_access deny !Safe_ports

    # Deny CONNECT to other than secure SSL ports
    http_access deny CONNECT !SSL_ports

    # Only allow cachemgr access from localhost
    http_access allow localhost manager
    http_access deny manager

    # We strongly recommend the following be uncommented to protect     innocent
    # web applications running on the proxy server who think the    only
    # one who can access services on "localhost" is a local user
    #http_access deny to_localhost

    #
    # INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
    #

    # Example rule allowing access from your local networks.
    # Adapt localnet in the ACL section to list your (internal) IP  networks
    # from where browsing should be allowed
    http_access allow localnet
    http_access allow localhost

    # And finally deny all other access to this proxy
    http_access deny all

    # Squid normally listens to port 3128
    http_port 3128

    # Uncomment and adjust the following to add a disk cache    directory.
    #cache_dir ufs /var/spool/squid 100 16 256

    # Leave coredumps in the first cache dir
    coredump_dir /var/spool/squid

    #
    # Add any of your own refresh_pattern entries above these.
    #
    refresh_pattern ^ftp:		1440	20%	10080
    refresh_pattern ^gopher:	1440	0%	1440
    refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
    refresh_pattern .		0	20%	4320
    ```

3. 直接啟動 squid

    ```bash
    /usr/sbin/squid start
    ```

4. 確認啟動

    ```bash
    netstat -tulnp | grep squid
    ```

    ![1start](https://user-images.githubusercontent.com/3851540/84596036-db3c3800-ae8d-11ea-8df1-cebaa8445270.jpg)

* 可設為開機啟動服務

    ```bash
    systemctl enable squid
    ```

## 使用方式

1. 一般存取網頁

    ```bash
    curl -L https://blog.yowko.com/
    ```

2. 確認透過 squid proxy 存取正常

    ```bash
    curl -x http://127.0.0.1:3128 -L https://blog.yowko.com/
    ```

3. 檢視 squid log 的存取紀錄

    > 預設位置 `/var/log/squid/access.log`

    ```bash
    cat /var/log/squid/access.log
    ```

    ![2accesslog](https://user-images.githubusercontent.com/3851540/84596038-dd05fb80-ae8d-11ea-9f42-3f629c7b1ef8.jpg)

## 設定 trasparnet mode

透過上面範例可以發現每次發送 request 都要指定 proxy，我相信便利性會大打折扣，下面就是透過將 squid 設定為 trasparnet mode 並搭配 nat 設定從網路層攔劫特定 request

1. 確定 os 層可以進行 ip forward

    > 避免開機後遺失，請修改 `/etc/sysctl.conf`

    ```bash
    sysctl -w net.ipv4.ip_forward=1
    sysctl -w net.ipv6.conf.all.forwarding=1
    sysctl -w net.ipv4.conf.all.send_redirects=0
    ```

2. 調整 config

    - 原始設定

        ```conf
        http_port 3128
        ```

    - 新設定

        > 3.0 之前使用 `transparent`，3.1 開始改用 `intercept`

        ```conf
        http_port 3128
        http_port 3129 intercept
        ```

3. 設定 iptables

    > 1. 避免 `squid` 服務會一直轉導至自己身上形成無窮迴圈
    > 2. 將 port 80 的 request 都改由 squid intercept 處理

    ```bash
    iptables -t nat -A OUTPUT -p tcp -m tcp --dport 80 -m owner --uid-owner squid -j RETURN
    iptables -t nat -A OUTPUT -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 3129
    ```

4. 直接 curl 就可以發現經由 squid proxy 處理

    ![3transparent](https://user-images.githubusercontent.com/3851540/84596039-dd05fb80-ae8d-11ea-813a-76c526ff039d.jpg)

## 心得

經由簡單的設定，現在有了一個簡易的 proxy server，不過這個 proxy server 在 transparent mode 下因為 https 關係會出現錯誤，接著我們再來看看如何設定 https 的解析吧 - [Squid Proxy Https 設定](https://blog.yowko.com/squid-proxy-https)

## 參考資訊

1. [Squid proxy - a short guide (forward & transparent proxy examples, SSL bumping, links to guides)](https://www.reddit.com/r/sysadmin/comments/a67hly/squid_proxy_a_short_guide_forward_transparent/)
2. [http_port](http://www.squid-cache.org/Versions/v3/3.5/cfgman/http_port.html)
3. [Squid Proxy Https 設定](https://blog.yowko.com/squid-proxy-https)
