---
title: "為 TortoiseGit 設定需驗證的 proxy"
date: 2016-12-30T00:42:34+08:00
lastmod: 2021-10-14T00:42:34+08:00
draft: false
tags: ["Tools","Git"]
slug: "tortoisegit-proxy"
aliases:
    - /2016/12/tortoisegit-proxy-with-authentication.html
---
## 為 TortoiseGit 設定需驗證的 proxy
 Windows 下進行 Git 相關操作， `TortoiseGit` 是非常便捷的工具，不僅操作介面和熟悉的 TortoiseSVN 大致相同，加上大多數的 git 指令都有實作，絕大多情境都可以透過 `TortoiseGit` 完成，而公司的網路環境需要 proxy 才能對外連線，順手紀錄一下相關設定方式

## 失敗訊息

- time out

    ![timeout](https://cloud.githubusercontent.com/assets/3851540/21704691/e2fcc814-d3f5-11e6-9e5c-84bf10af3c95.png)

## 在資料夾按右鍵開啟設定頁面

![menu](https://cloud.githubusercontent.com/assets/3851540/21704688/e2f56b14-d3f5-11e6-9274-a5598f70232f.png)

## Network 設定 proxy

1. 勾選 Enable Proxy Server
2. 依序填入
    - Server address
    - Port
    - Username
    - Password

        ![setting](https://cloud.githubusercontent.com/assets/3851540/21704689/e2f8fd88-d3f5-11e6-9b6f-dd2f77836b86.png)

3. 會將設定同步至 `.gitconfig` (位置在 `C:\Users\{username}\.gitconfig`)

## 成功存取

![success](https://cloud.githubusercontent.com/assets/3851540/21704690/e2fa368a-d3f5-11e6-8430-5cbf8cd3e83d.png)

## 注意問題

我沒有找到可以 bypass 內部網路的設定，所以外網跟內網都會吃同一組設定，即啟用 proxy  就是內網、外網同時啟用，可能啟用後內網就會有問題。
如果覺得很麻煩，可以試著把 proxy 指向 fiddler ，讓 fiddler 去處理這個問題

## 參考資訊

1. [TortoiseGit's Settings](https://tortoisegit.org/docs/tortoisegit/tgit-dug-settings.html)
