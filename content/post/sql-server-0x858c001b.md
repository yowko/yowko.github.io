---
title: "解決 SQL Server 安裝錯誤代碼 - 0x858C001B"
date: 2017-01-26T00:42:34+08:00
lastmod: 2021-10-26T00:42:34+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-0x858c001b"
aliases:
    - /2017/01/sql-server-0x858c001b.html
---
## 解決 SQL Server 安裝錯誤代碼 : 0x858C001B

雖然 DB 功能不怎樣，不過為了加速開發本機 SQL Server 到也安裝好幾次，今天遇到嚴重錯誤，紀錄一下原因

過去為了查問題方便 google 相關資訊都是安裝英文版，但是幾次問題下來發現 SQL Server 功能不熟悉之外再加上語言隔閡常找不到相關設定，所以改安裝中文，想不到就發生錯誤

## 錯誤訊息

1. 訊息內容

    ```log
    TITLE: SQL Server Setup failure.
    ------------------------------

    SQL Server Setup has encountered the following error:

    The SQL Server license agreement cannot be located for the selected edition, EXPRESS. This could be a result of corrupted media or the edition being unsupported by the media.

    Error code 0x858C001B.

    For help, click: http://go.microsoft.com/fwlink?LinkID=20476&ProdName=Microsoft%20SQL%20Server&EvtSrc=setup.rll&EvtID=50000&EvtType=0xFDC38F1F%25400xA40CEF17%25401420%254027

    ------------------------------
    BUTTONS:

    OK
    ------------------------------

    ```

2. 錯誤截圖

    ![1error](https://cloud.githubusercontent.com/assets/3851540/22196253/41af67ce-e187-11e6-992c-b37fac157595.png) 

## 問題原因

- Windows 語系 與 sql server 語系不一致

## 解決方式

1. 調整 Windows 語系與 Sql Server 版本相同
2. 使用與 Windows 語系相同的 Sql Server 進行安裝

## 參考資料

1. [SQL 2012 Express 安裝報錯 Error code 0x858C001B](http://blog.csdn.net/ccie38499/article/details/12784749)
