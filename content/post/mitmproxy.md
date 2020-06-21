---
title: "安裝 Mitmproxy"
date: 2020-06-20T21:30:00+08:00
lastmod: 2020-06-20T21:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "mitmproxy"
---

## 安裝 Mitmproxy

之前筆記 [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy) 提到為了加強與 Partner 間資料介接交換時的安全性，所以在 server 間會需要互相 trust ip，但這麼一來在業務增長時就會失去彈性，所以打算透過加入 proxy server 來進行 trust，讓後續可能新加入的 server 只要透過該 proxy server 就可以正常運作

嘗試過 `Squid`，設定方式可以參考 [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy) 與 [Squid Proxy Https 設定](https://blog,yowko.com/squid-proxy-https)，但因為 `Squid` 不支援 https 轉發 http，在使用上有限制，所以今天就來試試 `Mitmproxy`

說到 `Mitmproxy`，其實也不算陌生，之前在 [讓iOS 裝置可以存取自訂domain - Yowko's Notes](https://blog.yowko.com/ios-private-domain/) 就是透過這套工具來達成目的，而這次情境比較正式，改用安裝實體服務方式來紀錄

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. python 3.6.8
4. pipx 0.15.4.0
5. Mitmproxy v5.1.1

## 基本安裝

> 官方建議使用 binary 來安裝，但我到 [download 頁面](https://mitmproxy.org/downloads/) 提到可以用 `brew`, `pip`, `WSL`, or `Docker`

1. 安裝 python3

    > python 3.6 以上版本

    ```bash
    yum install -y python3
    ```

2. 安裝 pipx

    ```bash
    python3 -m pip install --user pipx
    python3 -m pipx ensurepath
    ```

    > 個人有重新登入套用新的環境變數

    ```bash
    pipx completions
    ```

3. 使用 pipx 安裝

    ```bash
    pipx install mitmproxy
    ```

## 使用方式

1. 啟動 mitmproxy

    >預設為 `8080`

    ```bash
    mitmproxy
    ```

    > `--listen-port` : 指定 proxy port 為 `9000`

    ```bash
    mitmproxy --listen-port 9000
    ```

2. 藉由 mitmproxy 存取 web page

    ```bash
    curl -x localhost:9000 -k -L https://blog.yowko.com
    ```

3. 測試結果

    ![1result](https://user-images.githubusercontent.com/3851540/85227747-d9d0b980-b411-11ea-9b7b-d0e0adaf61cd.jpg)

## 心得

Mitmproxy 在設定上比起 Squid 顯得更單純，不過這也不一定是好事，像是

1. 預設的 port、mode 都需要查文件才知道，沒辦法從 config file 中看出來
2. log 部份雖然比起 Squid 有更好的互動性跟更多資訊，但預設在 console 輸出對於正式服務還是沒那麼方便
3. https 驗證上會比嚴格， curl 使用時都需要加上 `insecure`

- log 處理方式請參考 [將 Mitmproxy log 存至檔案](https://blog.yowko.com/mitmproxy-log-to-file)

- https 請參考 [Mitmproxy 啟用 Https](https://blog.yowko.com/mitmproxy-https)

## 參考資訊

1. [mitmproxy](https://mitmproxy.org/)
2. [Installation](https://docs.mitmproxy.org/stable/overview-installation/#installation-from-the-python-package-index-pypi)
3. [安裝 Squid Proxy](https://blog.yowko.com/squid-proxy)
4. [Squid Proxy Https 設定](https://blog,yowko.com/squid-proxy-https)
5. [讓iOS 裝置可以存取自訂domain - Yowko's Notes](https://blog.yowko.com/ios-private-domain/)
6. [將 Mitmproxy log 存至檔案](https://blog.yowko.com/mitmproxy-log-to-file)
7. [Mitmproxy 啟用 Https](https://blog.yowko.com/mitmproxy-https)
