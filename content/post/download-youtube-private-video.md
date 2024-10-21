---
title: "下載 youtube 私人影片"
date: 2024-10-21T00:30:00+08:00
lastmod: 2024-10-21T00:30:31+08:00
draft: false
tags: ["bash","youtube"]
slug: "download-youtube-private-video"
---

## 下載 youtube 私人影片

陸陸續續上傳了一些家庭影片在 youtube，並設定成私人影片，原本的想法是透過 youtube 來備份影片，或是需要播放時可以隨時隨地開啟，不需要額外準備隨身碟，但實際情況卻是很容易遇到網路狀況不佳，或是需要播放時沒有網路的情況，加上原始檔案已經不知所蹤所以想要將私人影片下載下來。

如果是影片所屬帳號，後台管理介面可以直接下載，今天是紀錄非擁有者下載私人影片的方法。

事實上有不少軟體似乎可以下載 youtube 私人影片，不過有些限制 windows 平台，有些則需要付費，所以這邊使用 yt-dlp 來下載 youtube 影片，不過 yt-dlp 也有一些限制，例如私人影片，yt-dlp 會需要 youtube 的 cookie

## 基本環境說明

- macOS Sequoia 15.0.1 (Apple M2 Pro)
- yt-dlp_macos 2024.10.07
- [chrome extension: Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)

## 使用方式

1. 匯出 youtube cookie

    > 這只適用原本就有權限觀看的影片，如果沒有權限觀看的影片，這個方法不適用

    使用 [chrome extension: Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc) 來取得 youtube 的 cookie

    - 開啟 youtube 網站
    - 點擊 Get cookies.txt LOCALLY extension
    - 點擊 Export (會下載 www.youtube.com_cookies.txt)

        > 注意格式需為 `Netscape`

        ![savecookies](https://github.com/user-attachments/assets/2440f2d8-91a1-4d28-adec-197023e7e37d)

2. 使用 yt-dlp 下載 youtube 影片

    - 使用適合的 yt-dlp

        > [GitHub yt-dlp Release](https://github.com/yt-dlp/yt-dlp/releases) 依所在環境選擇適合的 yt-dlp

    - 使用 cookie 下載 youtube 影片

        - 語法

            ```bash
            {yt-dlp 執行檔} --cookies {cookie 檔案} {youtube 影片網址}
            ```

        - 範例

            ```bash
            ./yt-dlp_macos --cookies www.youtube.com_cookies.txt https://youtu.be/28vCvruM4oIOD
            ```

            > `28vCvruM4oI` 為 youtube 影片 id

            ![2download](https://github.com/user-attachments/assets/9c856b86-2a23-4098-8842-defe000a817a)

## 心得

我沒有嘗試下載套裝軟體，逐一嘗試很花時間，有試過一些線上工具，但沒有試到能用的

原本找不到方式下載 youtube 私人影片，還想過直接轉錄畫面，幸好後來發現 yt-dlp 可以透過 cookie 下載 youtube 影片

## 參考資料

1. [GitHub:yt-dlp/yt-dlp](https://github.com/yt-dlp/yt-dlp)
2. [chrome extension: Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)
