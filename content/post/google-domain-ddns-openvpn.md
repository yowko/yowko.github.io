---
title: "使用 Google Domains Dynamic DNS 搭配 OpenVPN 建立 VPN"
date: 2019-02-13T21:30:00+08:00
lastmod: 2019-02-13T21:30:31+08:00
draft: false
tags: ["Network"]
slug: "google-domain-ddns-openvpn"
---
# 使用 Google Domains Dynamic DNS 搭配 OpenVPN 建立 VPN
家人要前往內地進修幾個月，預計申請當地門號及網路服務，但又需要 facebook、google、line ....等服務，所以我就透過這個機會自行架設 VPN 來試試.....

## 基本環境說明
1. ASUS RT-AC68U (3.0.0.4.384_45149)
2. Google Domains

## Dynamic DNS 設定

> 讓動態 ip 也能設定 vpn，本次使用 [Google Domains](https://domains.google.com)

1. DNS --> Dynamic DNS --> 指定名稱 --> Add

    ![1ddns](https://user-images.githubusercontent.com/3851540/52728324-0a207080-2ff2-11e9-8e42-4afe669ab84c.png)

2. 取得認證資訊 

    > 等等在 Router 的設定會用到

    ![2getcred](https://user-images.githubusercontent.com/3851540/52728325-0ab90700-2ff2-11e9-9ee5-0080ee89bd22.png)

    ![3showcred](https://user-images.githubusercontent.com/3851540/52728328-0ab90700-2ff2-11e9-97ae-85703ef7c22b.png)

## Router (RT-AC68U) 設定
1. DDNS

    - 啟用 DDNS Client

        ![7ddns1](https://user-images.githubusercontent.com/3851540/52728319-0987da00-2ff2-11e9-8464-ec125b591fbd.png)

        ![8ddns2](https://user-images.githubusercontent.com/3851540/52728320-0987da00-2ff2-11e9-8dd8-71b6d1139c9a.png)

    - 使用 DOMAINS.GOOGLE.COM

        - a. `伺服器` 選擇 `DOMAINS.GOOGLE.COM`
        - b. `主機名稱` ：使用於 Google Domains 的設定值
        - c. `用戶名稱或 E-mail 帳號`：使用 Google Domains 產生的 Username
        - d. `密碼或 DDNS 金鑰`：使用 Google Domains 產生的 Password

            ![9ddns3](https://user-images.githubusercontent.com/3851540/52728321-0987da00-2ff2-11e9-82af-7da92684e2e9.png)

    - 套用本頁面設定後會產生 Server Certificate

        ![10ddns4](https://user-images.githubusercontent.com/3851540/52728322-0a207080-2ff2-11e9-9f89-2bb291037fc0.png)

2. VPN -  OpenVPN

    - 開啟 OpenVPN 伺服器

        ![4vpn1](https://user-images.githubusercontent.com/3851540/52728330-0ab90700-2ff2-11e9-880a-cc1136ae96a2.png)

        ![5vpn2](https://user-images.githubusercontent.com/3851540/52728996-456f6f00-2ff3-11e9-9fe2-994bf0e13179.png)
    
    - 設定用戶名稱與密碼
        
        ![6vpn3](https://user-images.githubusercontent.com/3851540/52728318-0987da00-2ff2-11e9-838d-6c672ef25326.png)

    - 匯出 OpenVPN 設定檔

        ![11export](https://user-images.githubusercontent.com/3851540/52728323-0a207080-2ff2-11e9-9e47-9698ec90a9ec.png)


## Client 設定

> 各個平台該如何使用匯出的 OpenVPN 設定檔來進行連線，可以參考以下連結

- [Windows](https://www.asus.com/tw/support/FAQ/1004469)
- [Mac OS](https://www.asus.com/tw/support/FAQ/1004472)
- [iPhone/iPad](https://www.asus.com/tw/support/FAQ/1004471)
- [Android](https://www.asus.com/tw/support/FAQ/1004466)

## 心得
雖然有自己找過資料，但問過經驗豐富的發哥果然還是最快方式，關鍵字一給馬上就找到正確設定方法，有可靠的朋友真好  感謝發哥

另外 Google Domains 在 DDNS 上與 Asus router 上的設定也非常友善，花得時間很少，滿意度很高，滿方便的

# 參考資訊
1. [動態 DNS](https://support.google.com/domains/answer/6147083?hl=zh-Hant)
2. [Google Domains](https://domains.google.com)

