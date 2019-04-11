---
title: "使用 fiddler 內建 proxy 來截錄手機或是程式封包"
date: 2017-03-05T01:42:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "fiddler-proxy-collect-traffic"
aliases:
    - /2017/03/use-fiddler-proxy-gather-traffic.html
---
# 使用 fiddler 內建 proxy 來截錄手機或是程式封包
某些情境下我們會使用 fiddler 來監控對外網路的 request 狀況，但有些情境下對外的 resource 是由程式或是 app 發動，fiddler 就無法直接取得相關資訊，今天就來紀錄一下平常我用來偵錯的方式：使用 fiddler 內建的 proxy 功能來蒐集並轉傳資訊

## fiddler 設定
1. fiddler 主選單 Tools --> Terelik Fiddler Options...
    
    ![1options](https://cloud.githubusercontent.com/assets/3851540/23497391/caffc318-ff5d-11e6-8553-6c122811cc50.png) 
2. Connections
    
    ![2connection](https://cloud.githubusercontent.com/assets/3851540/23497388/cafcacdc-ff5d-11e6-90d8-7a798689bb4d.png) 
    - 預設使用 `8888` port
    - Allow remote computer to connect
3. 設定完成需重新啟動 fiddler 才會生效


## 手機設定(以 android 為例)
1. 開啟 "設定"
    
    ![3androidsetting](https://cloud.githubusercontent.com/assets/3851540/23497389/cafe08f2-ff5d-11e6-9ed3-0e4f4d556173.png) 
2. 選擇 "wifi"
    
    ![4androidwifi](https://cloud.githubusercontent.com/assets/3851540/23497390/caff5478-ff5d-11e6-9da6-1b9424f28e1a.png) 
3. 在使用的 wifi 上長按
    
    ![5androidwifi](https://cloud.githubusercontent.com/assets/3851540/23497392/cb00efea-ff5d-11e6-9631-e89793d800a4.png) 
4. 修改網路
    
    ![6wifichange](https://cloud.githubusercontent.com/assets/3851540/23497393/cb052bdc-ff5d-11e6-9118-87c68fb267d3.png) 
5. 顯示進階選項
    
    ![7advance](https://cloud.githubusercontent.com/assets/3851540/23497394/cb210c94-ff5d-11e6-92e7-88ddf591dd6c.png) 
6. Proxy 設定選擇`手動`
    
    ![8munual](https://cloud.githubusercontent.com/assets/3851540/23497395/cb229c76-ff5d-11e6-99de-be03b1773b65.png) 
7. 填入資料
    - Proxy 主機名稱

        > 填開啟 fiddler 準備收資料的那台電腦名稱 or ip 
    - Proxy 通訊埠 

        > 填 fiddler 的 proxy port ，預設為 `8888`

    ![ip_proxy](https://user-images.githubusercontent.com/3851540/45498458-a37df900-b7ac-11e8-8798-948d19f7bcd0.png)
8. 開啟測試目標 (e.g. APP or 網頁)
    
    ![9result](https://cloud.githubusercontent.com/assets/3851540/23497396/cb24124a-ff5d-11e6-839b-d4d0450392d1.png) 
9.  fiddler 截取到目標
    
    ![10captured](https://cloud.githubusercontent.com/assets/3851540/23497397/cb254962-ff5d-11e6-9bb9-511247917f87.png)

## 心得
經過 fiddler 的 proxy 讓我們可以直接截錄到 app 或是 後端程式 實際送出什麼樣的資訊與內容，可以提供我們更多的訊息來 debug ，也可以透過 fiddler 的功能來調整傳送內容來進行測試(像是 postman 發出 request )