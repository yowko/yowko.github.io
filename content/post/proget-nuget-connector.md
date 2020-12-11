---
title: "透過 ProGet 取得官方 NuGet 套件"
date: 2018-01-13T23:57:00+08:00
lastmod: 2020-12-11T23:57:31+08:00
draft: false
tags: ["套件","NuGet","Tools"]
slug: "proget-nuget-connector"
aliases:
    - /2018/01/proget-nuget-connector.html
---
# 透過 ProGet 取得官方 NuGet 套件
之前筆記 [將 NuGet Package 發行至 ProGet](/2017/09/nuget-package-proget.html) 曾經介紹到如何將內部使用的 NuGet package 發行至內部 NuGet server：ProGet 上，在試行一段時間後決定擴大範圍，將所有 NuGet package 皆透過內部 ProGet 來取得，不再將所有 NuGet package commit 至 git 中

因為公司的各個環境皆屬封閉環境，對外皆未開通網路連線，僅能連到 NuGet server，所以需要將 NuGet package 下載至內部 ProGet 上，立馬來看看可以如何設定

## 一開始僅有內部 package

![1defaultproget](https://user-images.githubusercontent.com/3851540/34907623-820efb98-f8bc-11e7-9882-17b72cbd78e8.png)

## ProGet 設定官方 NuGet 連線

1.  建立 Connector
    *   Feeds --> Connectors --> Create Connector

        ![2createconnector](https://user-images.githubusercontent.com/3851540/34907624-8234bbf8-f8bc-11e7-9821-1460b275e3ea.png)

    *   Connector 相關資訊
        *   Feed type

            > 選擇 `NuGet`

        *   Connector URL

            > 使用 `https://www.nuget.org/api/v2`

        *   Connector name

            > 自訂識別用名稱

    ![3connectorinfo](https://user-images.githubusercontent.com/3851540/34907625-825a2d20-f8bc-11e7-9b46-5ae2c81e9f58.png)

2.  綁定 Connector
    *   Feeds --> Feeds --> {feed name}

        ![4feeds](https://user-images.githubusercontent.com/3851540/34907626-82808a56-f8bc-11e7-8049-b145835c840e.png)

    *   Manage Feed

        ![5mangefeed](https://user-images.githubusercontent.com/3851540/34907627-82a6fccc-f8bc-11e7-87b5-4bea757c8635.png)

    *   add connector

        ![6addconnector](https://user-images.githubusercontent.com/3851540/34907628-82ce70ea-f8bc-11e7-8702-49f0eedede37.png)

        ![7selectconnector](https://user-images.githubusercontent.com/3851540/34907629-82f4fc38-f8bc-11e7-98f7-ffbfac1670b1.png)

        ![8added](https://user-images.githubusercontent.com/3851540/34907620-819a63e6-f8bc-11e7-98b3-c34045bb7f15.png)

## 設定完成：包含內部及官方的 package

![9bothpackage](https://user-images.githubusercontent.com/3851540/34907621-81c22660-f8bc-11e7-8031-2dd03e443e67.png)

## 心得

設定步驟不多也不難，只是官方說明文件不是很清楚，無法照著操作就完成設定，還是需要自行摸索一番

另外就是 connector 的 health check 會有時間差，不是一設定完成就會執行，這點也容易造成誤會是不是沒設定成功，檢查是否設定成功最快的方式就是直接透過 Visual Studio 連線至內部 nuget feed 來查詢外部套件最快

![10healthcheck](https://user-images.githubusercontent.com/3851540/34907622-81e8a18c-f8bc-11e7-9bd5-4f5557d21fe9.png)

除此之外，ProGet 對應的 NuGet 還停留在 `NuGet V2 feeds` 版本，如果需要 v3 的相關功能要特別留意

# 參考資訊

1.  [將 NuGet Package 發行至 ProGet](/2017/09/nuget-package-proget.html)
