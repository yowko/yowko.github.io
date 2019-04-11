---
title: "為 Visual Studio Code (VSC) 設定需驗證的 proxy"
date: 2017-01-07T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["Tools","Visual Studio Code"]
slug: "visual-studio-code-proxy"
aliases:
    - /2017/01/visual-studio-code-vso-behind-proxy-with-authentication.html
    - /2017/01/visual-studio-code-proxy
---
# 為 Visual Studio Code (VSC) 設定需驗證的 proxy
Visual Studio Code 自發佈以來，受到不少開發人員的喜愛，更難能可貴的是非微軟的開發人員也給予不錯的評價，也逐漸成為大家開發時不可或缺的工具，就讓我們來看看該如何設定 proxy 吧！

## 1. File --> Preference --> User Settings

![VSO_SETP1](https://cloud.githubusercontent.com/assets/3851540/21706249/9ffee168-d400-11e6-98c5-4abf6158f73e.png)

## 2. VSC 會開啟分割視窗

![spiltwindows](https://cloud.githubusercontent.com/assets/3851540/21706248/9ffe9e2e-d400-11e6-9091-c56c63fcd404.png)

- 2-1. 左邊(`Default settings`) 

    > 所有設定的名稱及相關說明

- 2-2. 右邊(`setting.json`)

    > 客製設定內容   


## 3. 將 proxy 設定加入 setting.json

```json
{
    "http.proxy": "http://UserName:password@proxyserver:proxyport",
    "https.proxy": "http://UserName:password@proxyserver:proxyport",
    "http.proxyStrictSSL": false
}
```
![proxy_setting](https://cloud.githubusercontent.com/assets/3851540/21706250/a004e5ea-d400-11e6-8411-4b48447094e8.png)

# 參考資料
1. [User and Workspace Settings](https://code.visualstudio.com/Docs/customization/userandworkspace)
2. [Setting up Visual Studio Code](http://code.visualstudio.com/docs/setup/setup-overview)