---
title: "Mitmproxy 啟用 Https"
date: 2020-06-21T21:30:00+08:00
lastmod: 2020-06-21T21:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "mitmproxy-https"
---

## Mitmproxy 啟用 Https

之前筆記 [安裝 Mitmproxy](https://blog.yowko.com/mitmproxy) 提到 Mitmproxy 在存取 https 資源時會出現問題，雖然在 curl 使用時加上 `--insecure` 或是 `-k` 就可以避免問題，但是總覺得麻煩也不夠漂亮，所以順手紀錄解決方式囉

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. Mitmproxy v5.1.1

## 安裝步驟

1. 憑證 pem 轉為 crt

    cd ~/.mitmproxy
    openssl x509 -in mitmproxy-ca-cert.pem -inform PEM -out mitmproxy-ca-cert.crt

2. 安裝憑證

    ```bash
    update-ca-trust force-enable
    cp mitmproxy-ca-cert.crt /etc/pki/ca-trust/source/anchors/
    update-ca-trust extract
    ```

3. 啟動 mitmproxy

    ```bash
    mitmproxy
    ```

## 實際效果

1. 修改前：存取 https 網站有提示憑證未信任

    - 提示訊息

        ```txt
        [root@blogdemo ~]# curl -x localhost:8080 -L https://blog.yowko.com
        curl: (60) Peer's certificate issuer has been marked as not trusted by the user.
        More details here: http://curl.haxx.se/docs/sslcerts.html

        curl performs SSL certificate verification by default, using a "bundle"
         of Certificate Authority (CA) public keys (CA certs). If the default
         bundle file isn't adequate, you can specify an alternate file
         using the --cacert option.
        If this HTTPS server uses a certificate signed by a CA represented in
         the bundle, the certificate verification probably failed due to a
         problem with the certificate (it might be expired, or the name might
         not match the domain name in the URL).
        If you'd like to turn off curl's verification of the certificate, use
         the -k (or --insecure) option.
        ```

    - 異常截圖

        ![1nottrusted](https://user-images.githubusercontent.com/3851540/85227808-1a303780-b412-11ea-951c-06a6faead52b.jpg)

2. 修改後：直接正確存取

    ![2trusted](https://user-images.githubusercontent.com/3851540/85227809-1b616480-b412-11ea-8474-9a39f46a2aaa.jpg)

## 心得

設定上步驟不多，但我沒找到官方的說明頁面，加上大部份操作步驟我也不懂背後的機制，就不多做評論了，如果日後真的理解了再來補充吧

## 參考資訊

1. [mitmproxy 安装指南](https://zhuanlan.zhihu.com/p/88462295)
2. [安裝 Mitmproxy](https://blog.yowko.com/mitmproxy)
