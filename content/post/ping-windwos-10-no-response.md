---
title: "Ping 不到 Windwos 10 的電腦？！"
date: 2017-01-09T00:42:34+08:00
lastmod: 2021-10-29T00:42:34+08:00
draft: false
tags: ["Windows 10","Network"]
slug: "ping-windwos-10-no-response"
aliases:
    - /2017/01/Cannot-ping-windwos-10.html
---
## Ping 不到 Windwos 10 的電腦？！

Ping Windwos 10 的電腦無法正常回應？！

今天在測試 IIS Express allow remote access 時，首先讓電腦可以找到對方，再來看網站可不可正常運作
結果 `windows 10 ping windows 8.1` 毫不意外的正常回應
但是，出乎意料地 windows 8.1 ping windows 10 竟然得不到回應
直覺就是設定問題！！

最後找到的解法是要 **開啟`檔案及印表機分享`**，開啟方式有兩個，備忘一下出乎意料的解決方式

## 1. 從網路設定

- 1-1. 開啟網路設定

    ![neworksetting](https://cloud.githubusercontent.com/assets/3851540/21752146/7dce474a-d60e-11e6-830b-42556e117a5d.png)

- 1-2. 改變進階分享選項

    ![sharingoption](https://cloud.githubusercontent.com/assets/3851540/21752120/2acaecd8-d60e-11e6-89c5-ef0235c1dd88.png)

- 1-3. 開啟檔案及印表機分享

    ![turnon](https://cloud.githubusercontent.com/assets/3851540/21752121/2acdf004-d60e-11e6-988a-ac3fd41c0b31.png)

## 2. 從防火牆設定

- 1-1. 開啟防火牆

    ![WF.MSC](https://cloud.githubusercontent.com/assets/3851540/21752201/36bf6ac2-d60f-11e6-949d-0219c16a0437.png)

- 1-2. 在`輸入規則` 找到 `檔案及印表機分享`

    > 注意是 `ICMPv4-In` , profile 是 `public`

    ![rule](https://cloud.githubusercontent.com/assets/3851540/21752122/2acf4530-d60e-11e6-9ea2-b845cd2da3b8.png)

- 1-3. 啟用規則

    ![enable](https://cloud.githubusercontent.com/assets/3851540/21752125/2ad29b36-d60e-11e6-8dbc-9a4ee67e450e.png)

## 兩者設定是一致的

實際測試下來，發現兩者的設定是相同，修改其中一個設定後另一個設定顯示也會同步調整.

## 參考資料

1. [Enable Ping Reply and FTP Traffic in Windows 10 and Server](http://www.sysprobs.com/enable-ping-reply-and-ftp-traffic-in-windows-10-and-server)
