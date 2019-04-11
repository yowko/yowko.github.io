---
title: "為 npm 及 yarn 設定需驗證的 proxy"
date: 2016-12-07T00:42:34+08:00
lastmod: 2018-09-05T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "npm-yarn-proxy"
aliases:
    - /2016/12/npm-behind-proxy-with-authentication.html
---
# 為 npm 及 yarn 設定需驗證的 proxy
1. npm 錯誤訊息
    - 使用 npm 安裝套件時出現 `ETIMEDOUT` 的錯誤，錯誤訊息中有提示可能是 proxy 的問題
    
    ![Error](https://cloud.githubusercontent.com/assets/3851540/21705165/330c26f8-d3f9-11e6-9a2b-4a1d4ef6af45.png)
2. yarn 錯誤訊息
    - yarn 安裝時 Error 是 `ETIMEDOUT` 
    - **yarn 是使用 npm 設定**
    
        ![Error](https://cloud.githubusercontent.com/assets/3851540/21705166/332f9ec6-d3f9-11e6-80c1-676509d35b63.png)

# 1. 設定
- 1-1. command line(Cmd.exe)
    - 包含需要認證的 proxy
        
        ```
        npm config set proxy http://UserName:password@proxyserver:proxyport
        npm config set https-proxy http://UserName:password@proxyserver:proxyport
        ```
    
    - 認證資訊包含 domain
        
        ```
        npm config set proxy http://domain%5CUserName:password@proxyserver:proxyport
        npm config set https-proxy http://domain%5CUserName:password@proxyserver:proxyport
        ```
    - 實測後非必要設定(如無法成功時仍可嘗試)
        
        ```
        npm config set strict-ssl false
        npm config set registry "http://registry.npmjs.org/"
        ```

- 1-2. 直接修改 npm config("C:\Users\username\.npmrc")
     - 包含需要認證的 proxy 
        
        ```
        proxy=http://UserName:password@proxyserver:proxyport
        https_proxy= http://UserName:password@proxyserver:proxyport
        ```

    - 認證資訊包含 domain
        
        ```
        proxy=http://UserName:password@proxyserver:proxyport
        https_proxy= http://UserName:password@proxyserver:proxyport
        ```

- 實測後非必要設定(如無法成功時仍可嘗試)
        
        ```
        strict-ssl=false
        registry=http://registry.npmjs.org/
        ```

***存檔後若未生效，請重啟 command line***

# 2. 檢視設定結果
- 取得設定內容
    
    ```
    npm config get proxy
    npm config get https_proxy
    ```
- 實測後非必要設定(如無法成功時仍可嘗試)
    
    ```
    npm config get registry
    npm config get strict-ssl
    ```

# 3. 安裝套件時指定 proxy
```
npm --proxy http://UserName:password@proxyserver:proxyport install -g  packagename
```

# 4. 成功設定
![成功](https://cloud.githubusercontent.com/assets/3851540/21705167/334bb958-d3f9-11e6-8390-6d40ea7037a3.png)

# 參考資料
1. [官方說明(http-proxy)](https://www.npmjs.com/package/http-proxy)
2. [官方說明(npm-config)](https://docs.npmjs.com/cli/config)
3. [stackoverflow](http://stackoverflow.com/questions/25660936/using-npm-behind-corporate-proxy-pac)