---
title: "MacBook 遇到需啟用硬體虛擬化錯誤"
date: 2019-03-17T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Mac","Docker"]
slug: "macbook-bootcamp-enable-virtualization"
---
## MacBook 遇到需啟用硬體虛擬化錯誤

這幾天開啟電腦時，都會出現 Docker 的錯誤提醒，原以為是 Windows 上了新的 patch 後迫使 Docker for Windows 需要連帶更新而沒有留意(過去因為 Windows 更新而讓 Docker for Windows 掛掉的經驗可不在少數呀，依之前經驗常要 reset Docker for Windows 設定甚至是重灌 Docker for Windows 才能解決)，想說今天有空再來處理，結果今天定神一看竟然是個過去遇過的奇怪問題，所以順手紀錄一下

## 基本環境說明

1. Windows 10 1803 (OS Build 17134.648)
2. macOS Mojave 10.14.3

## 錯誤訊息

- 訊息內容

    ```log
    An error occcurred

    Hardware assisted virtualization and data execution protection must be enabled in the BIOS. See https://docs.docker.com/docker-for-windows/troubleshoot/#virtualization-must-be-enabled
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/54493388-6f75c300-490a-11e9-8e37-485e1055123c.png)

## 確認硬體虛擬化未開啟

開啟工作管理員 --> Performance --> CPU --> Virtualization

![2taskmgr](https://user-images.githubusercontent.com/3851540/54493389-6f75c300-490a-11e9-947f-afff4535ab4b.png)

## 解決方式

1. 將 MacBook 重新啟動並進入 macOS

    > 在右下角的 Boot Camp 圖示上按右鍵 --> 以 OS X 重新開機

    ![3bootcamp](https://user-images.githubusercontent.com/3851540/54493390-6f75c300-490a-11e9-96b3-fee03c920adf.png)

2. 系統偏好設定

    ![4systempreference](https://user-images.githubusercontent.com/3851540/54493391-700e5980-490a-11e9-8c78-8dd426ab1765.png)
3. 啟動磁碟

    ![5startdick](https://user-images.githubusercontent.com/3851540/54493392-700e5980-490a-11e9-809c-e8ea00be883f.png)
4. 選擇 BOOTCAMP --> 重新開機

    ![6restart](https://user-images.githubusercontent.com/3851540/54493393-700e5980-490a-11e9-8776-08512fa4299f.png)

## 實際效果

![7enable](https://user-images.githubusercontent.com/3851540/54493394-70a6f000-490a-11e9-9d39-bfe68b1051d4.png)

## 心得

- 問題發生原因：未知
- 為什麼透過 macOS 重新啟動 boot camp 可以解決問題：未知
- boot camp 隱含 virtualization 設定：未知

其實關於遇到的這個問題，我還是覺得毫無頭緒，猜測大概是 Windows 的 patch 對 macOS 或是 boot camp 有什麼影響，所以設定跑掉了，需要使用進入 macOS 再重新套用一次設定，但可能找不到正確解答了，畢竟想要重新這個問題也不知該如何動手XD   暫時就以解決問題為主囉

## 參考資料

1. [How to enable hardware virtualization on a MacBook?](https://superuser.com/a/717460)
