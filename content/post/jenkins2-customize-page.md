---
title: "Jenkins 如何自訂首頁 header"
date: 2017-04-18T23:30:00+08:00
lastmod: 2021-11-02T23:30:24+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins2-customize-page"
aliases:
    - /2017/04/jenkis2-customize-page.html
    - /jenkis2-customize-page
---
## Jenkins 如何自訂首頁 header

同事希望在 Jenkins 頁面的 header 加上一些常用連結甚至是系統訊息，以利資訊揭露，讓訊息不同步的情況可以獲得改善，紀錄一下該如何設定

## 開啟允許使用 html 語法

* Manage Jenkins --> Configure Global Security

    ![1security](https://cloud.githubusercontent.com/assets/3851540/25125158/6a6c891e-2460-11e7-9a77-16811ca2783d.png)

* Markup Formatter --> Safe HTML

    ![2maarkup](https://cloud.githubusercontent.com/assets/3851540/25125157/6a6bdbcc-2460-11e7-8bb7-aa57218e3673.png)

## 自訂首頁 header

* Manage Jenkins --> Configure System

    ![3configsystem](https://cloud.githubusercontent.com/assets/3851540/25125159/6a6f8baa-2460-11e7-9ec7-2fb16560eae8.png)

* System Message

  * `<javascript>`、`<style>`、`<link>` 都無法使用，僅能使用 inline style
  * 實際效果當然就得靠個人美感XD 但又少了 Bootstrap 的支援@@"

        ![4sysmsg](https://cloud.githubusercontent.com/assets/3851540/25125160/6a70fe72-2460-11e7-88d5-601606c97996.png)

  * 可以 preview

        ![5preview](https://cloud.githubusercontent.com/assets/3851540/25125161/6a75c970-2460-11e7-9446-30160d3f9702.png)

## 畫面效果

![6result](https://cloud.githubusercontent.com/assets/3851540/25125162/6a76ab6a-2460-11e7-9330-386ae198c41d.png)

## 心得

本來想把 Jenkins 畫面改得超有質感，也將大家常用資訊擺上來，但沒有 bootstrap 我想就別為難工程師了吧，就純粹提供資訊讓大家各憑本事了
