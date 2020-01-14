---
title: "從 mac 使用 Container Ip 直連 (更新版)"
date: 2020-01-14T22:20:00+08:00
lastmod: 2020-01-14T22:20:31+08:00
draft: false
tags: ["macOS","Docker","Container"]
slug: "mac-access-container-via-ip"
---

## 從 mac 使用 Container Ip 直連 (更新版)

之前筆記 [從 mac 使用 Container Ip 直連](https://blog.yowko.com/mac-access-container-ip-host) 紀錄到如何在 mac 上直接透過 container ip 存取相關服務，不過設定過程還需要動用到 Kubernetes，讓設定方式變得相當複雜，相對地使用上便利性也降低許多，不僅成為了我心中的疙瘩還久久無法忘懷，最近在嘗試解決網友問題時意外嘗試出可行的解決方案，紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.2
2. Docker Desktop for Mac community 2.1.0.5(40692)

## 設定方式

- 設定指令

    ```bash
    sudo ifconfig {網路 interface} {target container ip} netmask 255.255.255.255 alias
    ```

    1. 網路 interface

        > 依不同網路而有差異，ex: wifi 網路可能是 `en0`，有線網路可能就是 `lo0`

    2. container ip

        > 這是實際服務所在的 container ip

- 實際範例

    > 以下使用之前筆記 [使用 docker 建立 Redis Cluster - 更新版- Yowko's Notes](https://blog.yowko.com/redis-cluster-docker/) 多個 redis instnace 為例

    ```bash
    sudo ifconfig en0 172.19.0.2 netmask 255.255.255.255 alias
    sudo ifconfig en0 172.19.0.3 netmask 255.255.255.255 alias
    sudo ifconfig en0 172.19.0.4 netmask 255.255.255.255 alias
    sudo ifconfig en0 172.19.0.5 netmask 255.255.255.255 alias
    sudo ifconfig en0 172.19.0.6 netmask 255.255.255.255 alias
    sudo ifconfig en0 172.19.0.7 netmask 255.255.255.255 alias
    ```

## 心得

網路一直是非科系出身的我，很常出現無力感的眾多技術之一，儘管有自覺想要把網路學好但常常教學文章愈看愈糊塗XD

本篇筆記對比上篇筆記 [從 mac 使用 Container Ip 直連](https://blog.yowko.com/mac-access-container-ip-host)，難度實在是天差地遠，只是在當時我就是找不到更好的解決方式

雖然以結果來看，確實可以解決問題，但我仍然無法解釋為什麼這樣的做法可行，也無從比較與上篇筆記 [從 mac 使用 Container Ip 直連](https://blog.yowko.com/mac-access-container-ip-host) 的做法差異與優劣，可能有機會再請教大神同事了

## 參考資料

1. [從 mac 使用 Container Ip 直連](https://blog.yowko.com/mac-access-container-ip-host)
2. [使用 docker 建立 Redis Cluster - 更新版- Yowko's Notes](https://blog.yowko.com/redis-cluster-docker/)
3. [Redirect IP Address to localhost](https://www.santoshsrinivas.com/redirect-ip-address-to-localhost/)
