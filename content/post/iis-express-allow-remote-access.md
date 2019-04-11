---
title: "讓 IIS EXPRESS 的網站允許外部連接(allow remote access)"
date: 2016-12-09T00:42:34+08:00
lastmod: 2018-09-05T00:42:34+08:00
draft: false
tags: ["IIS Express"]
slug: "iis-express-allow-external-access"
aliases:
    - /2016/12/iis-express-allow-remote-access.html
    - /2016/12/iis-express-allow-external-access
---
# 讓 IIS EXPRESS 的網站允許外部連接(allow remote access)
讓 IIS EXPRESS 的網站可以被對本機以外提供服務

前端、行動裝置再加上愈來愈多的 IOT 相關應用出現，後端工程師最大的功能大概就是提供 API 了吧(^^||)
透過將開發用的 IIS EXPRESS 直接用來驗證結果是否合乎預期，就能減少頻繁地部署程式到實體 IIS 上的時間跟動作，又可以查更多bug(大誤)

利用這樣的方式讓在同網段的其他裝置可以直接連進來，甚至還可以直接 attach IIS EXPRESS 進行 debug，聽起來是不是很誘人？！

btw：這適合開發人員自行測試 debug 用，如果要跟其他裝置整合還是應該透過 IIS 部署

# 1. 設定 HTTP.sys (Hypertext Transfer Protocol Stack )
> Note: 關於 `HTTP.sys`

> - IIS 6 時開始用來取代 Windows Sockets API
> - HTTP.SYS 是 IIS 的核心元件，用來處理 Http request
> - 在 IIS 建立新應用程式集區(application pool)時，會自動在 `http.sys` 註冊集區以用來識別.

-  1-1. 以管理者身份開啟命令提示字元(`command line`)
    
    ![cmd](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/377x256/d5f3403d2adf9fa5eea5f66fbdd676cf/commandline_win10_%E7%BB%93%E6%9E%9C.png)

- 1-2. 取得`本機內網 ip`
	- ipconfig
        
        ![ip](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/855x196/1a5927d40ba782da8e4a340f6bef17e7/localip_%E7%BB%93%E6%9E%9C.png)

- 1-3. 取得 `IIS EXPRESS 站台 PORT`
    - 方法1:專案設定
        
        ![port2](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1200x534/b53efbbdcae6cceaf9eafe62b85d41dc/port2_%E7%BB%93%E6%9E%9C.png)

    - 方法2:IIS EXPRESS 快顯視窗中
        
        ![port1](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/454x153/fb472d9acf3f9ae891dca7dda4a55229/port_%E7%BB%93%E6%9E%9C.png)

	
- 1-4. 使用 `netsh` 以`本機 IP` 及 `站台 port` 加入 URL 保留區(access control list)
	
    ```
    netsh http add urlacl url=http://192.168.31.102:10777/ user=everyone
    ```
    
    ![added](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/988x651/03619f68b409c9169c250b41cd0861f8/added_%E7%BB%93%E6%9E%9C.png)

- 1-5. 檢查設定(optional)

    ``` 
	netsh http show urlacl
    ```

- 刪除設定(if need)

    >如果已經測試完畢，才需要進行刪除

    ```
	netsh http delete urlacl url=http://192.168.31.102:10777/
    ```

# 2. 修改 `IIS EXPRESS` 設定
- 2-1. IISEXPRESS 8.X (VS 2013) 以前
    
    ``` 
	%USERPROFILE%\Documents\iisexpress\config\applicationhost.config
    ```
-  2-2. IIS Express 10(VS 2015)
    
    >改放在專案下的 `.vs\config` 資料夾中

    ```
	%USERPROFILE%\Documents\Visual Studio 2015\Projects\TestExceptional\.vs\config\applicationhost.config
    ```
- 2-3. 加上 binding
	
    >用 port 搜尋，複製現有的，把 `localhost` 改為 `本機內網 IP`
    
    ```
    <binding protocol="http" bindingInformation="*:10777:192.168.31.102" />
    ```
    
    ![binding](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1200x153/5f416fa4c798685e6023faf4e5e2ceae/binding_%E7%BB%93%E6%9E%9C.png)


# 3. 開放防火牆的特定連接埠(port)
- 3-1.`開始` 搜尋 `wf.msc`
    ![wf.msc](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/518x870/30fe96e7eba4b7fc141808af1145c205/wf1_%E7%BB%93%E6%9E%9C.png)

- 3-2. 新增 `inbound` 規則
    
    ![newrule](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1200x316/a9cc253b079013c62e96e664bf83b855/wf2_%E7%BB%93%E6%9E%9C.png)

- 3-3. 建立 `port` 規則
    
    ![PORT](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1070x871/5195ba38d5ed3af81b5c617a946b1280/wf3_%E7%BB%93%E6%9E%9C.png)

- 3-4. 使用 `TCP` 及 `特定 port`
    
    ![TCP](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1070x871/c94b260789f924452eeaedf47c9e8f64/wf4_%E7%BB%93%E6%9E%9C.png)

- 3-5. 選擇 `allow the connect`
    
    ![ALLOW](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1070x871/1c13924829503fed0f04189c81c4d80c/wf5_%E7%BB%93%E6%9E%9C.png)

- 3.6. 預設即可
    
    ![WHEN](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1070x871/5aeb39f4e1a5afd515306b52eb6800ab/wf6_%E7%BB%93%E6%9E%9C.png)

- 3-7. 建立規則名稱
    
    ![NAME](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1070x871/aeb5d48450ae2e56b694bc92f2c80178/wf7_%E7%BB%93%E6%9E%9C.png)


# 4. 效果
- 修改前(無法存取 IIS EXPRESS 站台)
    
    ![cannotaccess](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1003x659/3c885a15dfcf49cea3332155d360df17/cannot_%E7%BB%93%E6%9E%9C.png)
- 修改後(可以存取 IIS EXPRESS 站台)
    
    ![access](https://trello-attachments.s3.amazonaws.com/580596fb744dfbc732854015/1200x534/5af8d857e938fd2436132de04c8d59ca/ok_%E7%BB%93%E6%9E%9C.png)

# 參考資料
1. [MSDN http https](https://msdn.microsoft.com/zh-tw/library/ms733768.aspx)
2. [MSDN-IIS EXPRESS](https://msdn.microsoft.com/zh-tw/library/58wxa9w5%28v=vs.120%29.aspx?f=255&MSPPError=-2147217396)
3. [Nestsh commands for HTTP](https://msdn.microsoft.com/en-us/library/windows/desktop/cc307236.aspx)
4. [VS2015的IISEXPRESS 10的APPLICATIONHOST.CONFIG置叨位](http://blog.kkbruce.net/2015/07/where-vs2015-iisexpress-10-applicationhostconfig.html)
5. [Access IIS Express from another machine ](http://bendetat.com/access-iis-express-from-another-machine.html)


