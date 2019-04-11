---
title: "為 Charles  設定需驗證的 proxy"
date: 2017-01-03T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "charles-proxy"
aliases:
    - /2017/01/charles-proxy-with-authentication.html
---
# 為 Charles 設定需驗證的 proxy
`Charles` 是一套 HTTP proxy 、 HTTP monitor 、 Reverse Proxy 工具，可以讓開發人員補捉 http 封包來進行偵錯，在跨網路的開發上是必備的工具，功能上與 [fiddler](FIddler2.com) 非常類似。`Charles` 需付費使用( 30 天試用期, 30 天過後 隔一陣子時間會要求重啟)、`fiddler` 是免費軟體;`Charles` 有 Windows 跟 mac 兩個版本、`fiddler` 原本只有 Windows 版(mac 使用只能透過 mono)，<span style='color:red'>2016/10/17 已推出 Fiddler for OS X Beta 1</span>

## 1. 無法連線
- Status:Failed; 出現 Connection timed out.
    
    ![fail](https://cloud.githubusercontent.com/assets/3851540/21677662/0a99e94a-d376-11e6-9396-66e3bdd8dda6.png)

## 2. 設定 proxy

- 2-1. 開啟 `External Proxy Settings...`
    - Proxy --> External Proxy Settings...
        
        ![setting](https://cloud.githubusercontent.com/assets/3851540/21677661/0a975d10-d376-11e6-8166-933c1ecdeeaf.png)

- 2-2. 設定 `Web Proxy(HTTP)`
1. 勾選 Web Proxy(HTTP)
2. 依序填入 `Web Proxy Server`,`Proxy Server Port`,`Domain`,`Username`,`Password`
    
    ![setting](https://cloud.githubusercontent.com/assets/3851540/21677660/0a972eee-d376-11e6-8dcc-ac72308bbcd4.png)

- 2-3. 設定 `Secure Web Proxy(HTTPs)`
1. 勾選 Secure Web Proxy(HTTPs)
2. 依序填入 `Secure Web Proxy Server`,`Proxy Server Port`,`Domain`,`Username`,`Password`
    
    ![setting](https://cloud.githubusercontent.com/assets/3851540/21677658/0a5794aa-d376-11e6-8932-4a5ef093fc91.png)

## 成功連線
- Response Code :200
    
    ![ok](https://cloud.githubusercontent.com/assets/3851540/21677659/0a7b0e80-d376-11e6-8e5b-e5e332447bf4.png)


# 參考資料
1. [Charles](https://www.charlesproxy.com/)
2. [Fiddler與Charles的特殊用途](http://www.cnblogs.com/cos2004/archive/2013/04/17/3024171.html)
3. [Introducing Fiddler for OS X Beta 1](http://www.telerik.com/blogs/introducing-fiddler-for-os-x-beta-1)