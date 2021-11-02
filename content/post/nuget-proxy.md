---
title: "為 NuGet 設定需驗證的 proxy"
date: 2017-02-05T00:42:34+08:00
lastmod: 2021-11-01T00:42:34+08:00
draft: false
tags: ["Visual Studio","Visual Studio 2015","NuGet"]
slug: "nuget-proxy"
aliases:
    - /2017/02/visual-studio-2015-nuget-behind-proxy-with-authentication.html
---
## 為 NuGet 設定需驗證的 proxy

公司電腦密碼三個月就需要更換一次，連帶著代理伺服器 (proxy) 設定也需要更新一輪，但這次卻有點不同 - 我的 Visual Studio 2015 無法從 NuGet 取得相關 package。我一直記得我剛進公司時，將所有需要設定 proxy 的軟體，把相關設定步驟都紀錄下來了，但這次卻出乎我意料，我想一定是我在做其他測試時動到了相關設定，立馬來看看發生了什麼事吧

## 其他功能一切正常

1. VS 起始畫面 - 新聞

    ![1NEWS](https://cloud.githubusercontent.com/assets/3851540/22542520/27f17876-e967-11e6-926e-f43e390191c1.png)

2. VS Tools - Extensions and Updates

    ![2extension](https://cloud.githubusercontent.com/assets/3851540/22542521/282f702c-e967-11e6-9568-800876d9ddde.png)

## NuGet Package Manager 無法連線

![3nugeterror](https://cloud.githubusercontent.com/assets/3851540/22542522/2836ec6c-e967-11e6-8c01-8caf4923a462.png)

## NuGet Proxy 設定

1. 手動設定
    - 1-1. 開啟 Package Manager Console

        ![5nugetconsole](https://cloud.githubusercontent.com/assets/3851540/22542524/28751712-e967-11e6-8a62-c7ceb77c9c69.png)

    - 1-2. 依序設定 proxy
        - porxy server

            ```cmd
            nuget config -set http_proxy=http://proxyserver:port
            ```

        - username

            ```cmd
            nuget.exe config -set http_proxy.user=AD\username
            ```

        - password

            ```cmd
            nuget.exe config -set http_proxy.password=password
            ```

    - 1-3. 相關設定會儲存至 `C:\Users\{username}\AppData\Roaming\NuGet\NuGet.Config`

        ![4nugetconfig](https://cloud.githubusercontent.com/assets/3851540/22542523/2872541e-e967-11e6-96fa-30865c8c758a.png)

    - 1.4. 設定完成需重啟 VS
    - <span style="color:red">密碼會加密，需透過 Package Manager Console 設定</span>

2. 使用系統 Proxy
    - 將 NuGet.config 中關於 proxy 的設定清空，vs 即自動套用系統 proxy 設定

## 參考資料

1. [NuGet Behind Proxy](http://stackoverflow.com/questions/9232160/nuget-behind-proxy)
