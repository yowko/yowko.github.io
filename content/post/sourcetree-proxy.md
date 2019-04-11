---
title: "為 SourceTree 設定需驗證的 proxy"
date: 2017-01-08T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["Tools","Git"]
slug: "sourcetree-proxy"
aliases:
    - /2017/01/sourcetree-behind-proxy-with-authentication.html
---
# SourceTree 設定需驗證代理伺服器(proxy with authentication)
 SourceTree 是套 GUI 介面相較於 TortoiseGIT 有較多正面評價的 GIT 管理工具，相較於 TortoiseGIT 只有 Windows 版本，SourceTree 有 Windows 跟 MAC 版本，但不變的還是公司網路環境仍需要經由 proxy，就來看看該如何設定吧

## 無法連線
- 錯誤訊息：`Time out`
    
    ![timeout](https://cloud.githubusercontent.com/assets/3851540/21706337/641fa0dc-d401-11e6-8fa7-ab2b34322eb8.png)


## 設定 proxy
1. menu
    - Tools --> Options
        
        ![menu](https://cloud.githubusercontent.com/assets/3851540/21706343/6442a8f2-d401-11e6-97ab-393f2454d0c8.png)

2. Network
    
    ![network](https://cloud.githubusercontent.com/assets/3851540/21706341/6426800a-d401-11e6-9d4c-44b3a59a6a5a.png)

3. Proxy Setting
    - 勾選 `Use custom proxy settings`
    - 勾選 `Add proxy server configuration to Git/Mercurial` (這個選項會將設定寫至 `.gitconfig`- 位置在 `C:\Users\{username}\.gitconfig`)
    - 填入 `Server`、`Port`
        
        ![proxysetting](https://cloud.githubusercontent.com/assets/3851540/21706340/642619a8-d401-11e6-821b-c151b00e4923.png)

4. 設定帳號密碼
- 勾選`Proxy server requires username and password`
    
    ![account](https://cloud.githubusercontent.com/assets/3851540/21706342/6426c1dc-d401-11e6-8bce-e2d5199ecd0b.png)
- 設定帳號密碼
    
    ![password](https://cloud.githubusercontent.com/assets/3851540/21706338/64231e10-d401-11e6-8248-117f6877928a.png)

## 成功連線
![success](https://cloud.githubusercontent.com/assets/3851540/21706344/6446a0e2-d401-11e6-9d0d-ba7af101f7d1.png)

## 注意事項
- 如果帳號或密碼中有`@`，就無法從介面直接設定，可以進到 `.gitconfig` 中直接設定

- `.gitconfig` 位置在 `C:\Users\{UserName}`
    - 原密碼：`@password`   
    - 轉換後密碼：`%40password`
        
        ```
        [http]
            proxy = http://username:%40password@proxyserver:proxyport
        
        [https]
            proxy = http://username:%40password@proxyserver:proxyport
        ```
        ![gitconfig](https://cloud.githubusercontent.com/assets/3851540/21706339/6424c9ae-d401-11e6-91c0-10252b99fcb3.png)

# 參考資料
1. [Escape @ character in git proxy password](http://stackoverflow.com/questions/6172719/escape-character-in-git-proxy-password?noredirect=1&lq=1)