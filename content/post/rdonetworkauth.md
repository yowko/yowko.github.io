---
title: "RDO (Remote Desktop Organizer) 連線時出現需要 Network Level Authentication"
date: 2018-08-09T02:40:00+08:00
lastmod: 2021-10-28T02:40:52+08:00
draft: false
tags: ["Tools","Windows Server 2016"]
slug: "rdo-network-level-authentication"
---

## RDO (Remote Desktop Organizer) 連線時出現需要 Network Level Authentication

最近的專案需要整合外部廠商的相關功能，為避免影響其他服務就建立了一個全新的 Windows Server 2016  VM 來進行整合開發及測試，但 VM 建立後就一直無法透過 RDO (Remote Desktop Organizer) 直接遠端桌面進到 Windows Server 2016 中，就來看看該如何設定吧

## 錯誤訊息

1. 訊息內容

    ```log
    The remote computer requires Network Level Authentication, which your computer does not support. For assistance, contact your system administrator or technical support.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/43934466-11a6c95c-9c82-11e8-9056-aa160c3862d7.png)

## 解決方式

1. 取消連線目標機器的 Network Level Authentication

    > 詳細內容請參考 [The remote computer requires Network Level Authentication](https://www.thewindowsclub.com/the-remote-computer-requires-network-level-authentication)

    * 搜尋 `Remote Desktop settings`

        ![2remotesetting](https://user-images.githubusercontent.com/3851540/43934467-11ed2e2e-9c82-11e8-90eb-61f7b4bd7553.png)

    * 開啟 `Advanced settings`

        ![3advanced](https://user-images.githubusercontent.com/3851540/43934468-12182750-9c82-11e8-95b3-8233cd67cf17.png)

    * 關閉 `Require computers to use Network Level Authentication to connect (recommended)`

        ![4turnoff](https://user-images.githubusercontent.com/3851540/43934469-1243748c-9c82-11e8-9fa2-112f5433cb38.png)

2. 修改 RDO 的連線設定
    * 目標連線右鍵 --> Edit...

        ![5edit](https://user-images.githubusercontent.com/3851540/43934471-126c283c-9c82-11e8-92ec-cfd9dca438fc.png)
    * Advanced --> Enable Credential Security Support Provider (CredSSP)

        ![6credssp](https://user-images.githubusercontent.com/3851540/43934472-12951b02-9c82-11e8-9ef2-4f6028eca5e0.png)  

## 心得

原本以為是 Windows Server 2016 的安全性比較高的關係，但後來使用 Windows 10 連線一樣有相同問題，靈機一動使用 Windows 內建 Remote Desktop Connection (mstsc.exe) 竟可以正常順利連線，這才發覺可能是 RDO 設定問題造成的

雖然只是個小小設定，不過因此就不需調整目標機器上的 Network Level Authentication 建議設定，個人認為這才是正統做法：如果可以應該是調整連線啟動端而不是連線目標端

## 參考資料

1. [The remote computer requires Network Level Authentication](https://www.thewindowsclub.com/the-remote-computer-requires-network-level-authentication)
