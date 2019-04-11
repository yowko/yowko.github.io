---
title: "使用 ngrok 讓本機上的網站讓全世界看到"
date: 2016-12-19T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "ngrok"
aliases:
    - /2016/12/ngrok.html
---
# 使用 ngrok 讓本機上的網站讓全世界看到
這個 title 會不會下得太過火？！

回到主要目的，您一定有過這樣的經驗：想測試個功能，卻要大費周章地將程式部署到測試機、發揮創意做的小東西都沒有對外公開的 host 環境可以測試，即使是使用雲端，也是需要經過一些申請跟部署，加上費用也是考量的因素之一，而常常花在這些零碎工作的時間可能都比真正開發的時間要長。

現在我們可以使用 ngrok 這個服務來解決問題：我們可以將自己電腦上的網站透過 ngrok 的對應，讓其他人可以透過 ngrok 提供的網址來瀏覽原本只存在本機上的網站。

看完說明有沒有覺得是個好東西，就讓我們馬上開始吧

# 1. 如何開始
直接至 [下載點](https://ngrok.com/download) 下載對應 os 的版本並解壓縮

![1download](https://cloud.githubusercontent.com/assets/3851540/22231642/b623eaa6-e21f-11e6-965b-d1fc05cd0c17.png)


- ***如果連網需要透過 proxy 才需要進行下列步驟***

    ![2noconnect](https://cloud.githubusercontent.com/assets/3851540/22231646/b62ad154-e21f-11e6-845d-68847976693b.png)

    1. 先執行下列指令會自動建立 configuration 檔案
        
        ```
        ngrok authtoken {token}
        ```
    2. authtoken 需經由註冊取得 
        - [ sign up for an account](https://dashboard.ngrok.com/user/signup)
             
             ![3AUTHTOKEN](https://cloud.githubusercontent.com/assets/3851540/22231645/b6284c4a-e21f-11e6-836d-1694f0276349.png)


    3. 設定 proxy
        - 開啟 configuration 檔案
            
            >Windows 預設 configuration 路徑：`C:\Users\{username}\.ngrok2\ngrok.yml`
            
        - 加入 proxy 設定
            
            ```
            http_proxy: "http://user:password@proxyserver:port"
            ```
- ***如果連網需要透過 proxy 才需要進行上述步驟***

# 2. 執行
    
```
ngrok http port
```
> e.g. `ngrok http 80` 表示使用 http  80 port
    
- 預設同時啟用 `http` 及 `https`
    
    ![4enabletunnel](https://cloud.githubusercontent.com/assets/3851540/22231647/b6470d6a-e21f-11e6-8f4f-5fadd4b60d6c.png)

# 3. 取得可連線的 url
- 使用 ngrok
    
    ![5accessurl](https://cloud.githubusercontent.com/assets/3851540/22231648/b6479bcc-e21f-11e6-9c90-0c297b86b89e.png)

- 使用 Web Interface
    - 服務開啟後，預設會建立 Web Interface 來提供相關資訊
    - 預設是 `http://127.0.0.1:4040/`
        
        ![6accessurl](https://cloud.githubusercontent.com/assets/3851540/22231650/b649523c-e21f-11e6-8438-8472224186aa.png)

# 4. 即時偵測網站資源被呼叫狀況
會列出網站相關資料被存取的紀錄，及使用的 http verb

- 使用 ngrok
    
    ![7request](https://cloud.githubusercontent.com/assets/3851540/22231649/b648f706-e21f-11e6-8dcf-7fcb423d4a23.png)

- 使用 Web Interface
    
    ![8request](https://cloud.githubusercontent.com/assets/3851540/22231639/b5e37836-e21f-11e6-9426-080e2a03608e.png)

# 5. 檢視 request 及 response 細節

- 可以確認收到的 header 與 body 是否正確
    
    ![10requestresponse](https://cloud.githubusercontent.com/assets/3851540/22231643/b624c8a4-e21f-11e6-8bf2-f6edf84cd9a5.png)

# 6. 重新執行 request

![9replay](https://cloud.githubusercontent.com/assets/3851540/22231640/b605fabe-e21f-11e6-9fe4-216bd83e4721.png)


# 7. 加上網址存取密碼
    
```
ngrok http -auth="username:password" 80
```
- 設定`帳號` 及`密碼`，加強資安控管
    
    ![11password](https://cloud.githubusercontent.com/assets/3851540/22231644/b6262988-e21f-11e6-90d9-187ecc42e8aa.png)


# 8. 其他一堆設定......

> 請大家上 [ngrok 文件](https://ngrok.com/docs) 更深入瞭解

# 實際效果
可以由 ngrok 動態提供的 url access，結果與 localhost 網站完全相同

![12result](https://cloud.githubusercontent.com/assets/3851540/22231641/b622a45c-e21f-11e6-9788-7c75c14a6740.png)


# 參考資料
1. [ngrok 官網](https://ngrok.com/)
2. [ngrok 文件](https://ngrok.com/docs)