---
title: "LINQPad 5 無法連線 Oracle？！"
date: 2017-11-04T01:38:00+08:00
lastmod: 2021-11-02T01:38:07+08:00
draft: false
tags: ["Oracle","Tools"]
slug: "linqpad-5-oracle"
aliases:
    - /2017/11/linqpad-5-oracle.html
---
## LINQPad 5 無法連線 Oracle？!

LINQPad 的便利，想必許多 .Net 開發人員都感同身受，從以前用來撰寫 LINQ，到現在不僅可以用來驗證 C# 語法正確性，甚至還可以從 NuGet 安裝套件，功能非常強大

有時需要驗證專案中部份功能，從開啟專案到執行至特定位置常常要耗掉許多時間，還得經過重重驗證及特定流程，步驟相當繁瑣，一旦執行後發現語法錯誤需要調整，又得全部重新來過，無形中浪費很多時間，這時候我就會選擇將需要反覆調整的語法使用 LINQPad 開發，以加快開發及偵錯效率

想不到今天使用 LINQPad 連線至 Oracle 驗證 LINQ 語法時卻無法順利連線，搞定後紀錄一下做法

## 傳統操作(會出現錯誤)

1. 加入連線

    ![2addconnection](https://user-images.githubusercontent.com/3851540/32387380-44be5152-c0ff-11e7-83ec-e97f92451724.png)

2. 填寫連線資訊

    * 指定有 EntityFramework 連線的 dll
    * 指定 dll 中的 DBContext 完整 Type 名稱
    * 指定連線字串的 config 檔
    * 指定連線字串名稱

    ![3connectioninfo](https://user-images.githubusercontent.com/3851540/32387381-44f15a3e-c0ff-11e7-9ef0-e889dd85ef43.png)

3. 測試連線

    ![4testconnect](https://user-images.githubusercontent.com/3851540/32387382-4525e31c-c0ff-11e7-934b-fa35be5d9498.png)

4. 錯誤訊息
    * 訊息內容

        ```log
        An error occured processing the request.

        ConfigurationErrorsException: Failed to find ot load the
        registered .Net Framework Data Provider.
        ```

    * 錯誤截圖

         ![1error](https://user-images.githubusercontent.com/3851540/32387379-44862534-c0ff-11e7-99ad-ea9309751ce7.png)

## 解決方式

**使用 IQ Driver - for MySQL, SQLite, Oracle**

* 安裝 `IQ Driver - for MySQL, SQLite, Oracle`

  * View more drivers...

    ![5install1](https://user-images.githubusercontent.com/3851540/32387459-735d347e-c0ff-11e7-9b73-922c211fe329.png)

  * IQ Driver - for MySQL, SQLite, Oracle --> Download & Enable driver

    ![6install2](https://user-images.githubusercontent.com/3851540/32387461-738ca704-c0ff-11e7-9e94-e6fac20f0b11.png)

  * 完裝功成會出現 `IQ(Supports Oracle, MySQL, SQLite)`

    ![7installed](https://user-images.githubusercontent.com/3851540/32387454-72a40e9a-c0ff-11e7-8ca1-273bc96c66ee.png)

* 設定 Oracle 連線

    ![11oracleconnect](https://user-images.githubusercontent.com/3851540/32387778-854ec28c-c100-11e7-9a40-6923352ad40d.png)

    ![8connectinfo](https://user-images.githubusercontent.com/3851540/32387455-72d1dcf8-c0ff-11e7-922f-13f5b037c7f5.png)

    ![9success](https://user-images.githubusercontent.com/3851540/32387456-7300aaec-c0ff-11e7-86bc-4d66f3211acc.png)

    ![10ok](https://user-images.githubusercontent.com/3851540/32387458-732fc78c-c0ff-11e7-99eb-892e2fd5b97e.png)

## 心得

可能是被 MS-SQL Server 慣壞了，從轉換至 Oracle 後總覺得開發效率大降，不僅許多工具的支援度差強人意，就連使用方式常常都讓人覺得像是 hack 手法，穩定性跟可靠性都不足，不過可能是因為還不夠熟悉造成先入為主的排斥，希望隨著使用經驗增加可以少踩些雷 @@"

## 參考資訊

1. [LINQPad Supplementary Data Context Drivers](http://www.linqpad.net/richclient/datacontextdrivers.aspx)
