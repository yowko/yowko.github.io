---
title: "Redis 的安全設定"
date: 2017-03-03T01:42:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Redis"]
slug: "redis-security"
aliases:
    - /2017/03/redis-security.html
---
# Redis on Windows 的安全設定
redis 為了強調高性能表現及簡易使用方式，而將安全性前提建立於信任環境的信任用戶，所以未妥善設定的 redis 會直接赤裸裸地曝露在網路上，嚴重影響資安風險。為了避免機敏資料被盜取或是被刪除，一定要做些基本防護。

今天主要是用 Windows 上的 Redis 當做範例，大部份設定是相同的


## 設定方式
- 直接修改 redis 的組態 config 檔
    
    >`{安裝路徑}\Redis\redis.windows-service.conf`
- 組態 config 檔案位於 redis 安裝目錄下
- 修改後需重新啟動 service

## 安全設定
1. 預設啟用保護模式
    
    >protected-mode yes
    - redis 3.2.0 以後預設啟用
    - 保護模式啟用下，`且`符合下列兩條件時，僅允許本機存取
        1. 未限制網路存取
        2. 未設定密碼保護
    - 可以手動關閉
        
        >`protected-mode no`

2. 限制網路存取
    
    >bind {ip}
    
    - e.g. `bind 127.0.0.1`
    - 未加入 `127.0.0.1` 會導致 service 無法啟動
        
        ![1serviceerror](https://cloud.githubusercontent.com/assets/3851540/22244155/5cc179ce-e265-11e6-8112-09307879990b.png) 

3. 密碼驗證
    
    >requirepass {password}
    
    - 明碼儲存
    - 非加密傳輸
    - 建議使用較長密碼
    - e.g. `requirepass password`
        ![2connect](https://cloud.githubusercontent.com/assets/3851540/22244156/5cc5120a-e265-11e6-9c11-a466ca3a701e.png) 
    
        ![3success](https://cloud.githubusercontent.com/assets/3851540/22244157/5cc9c44e-e265-11e6-9f98-cb93269552be.png)

4. 使用指令別名
    
    >rename-command CONFIG {newname}
    
    - 將 `config` 指令改為 {newname} 
    - 避免一般使用者直接修改 redis 的設定
        
        >e.g. `rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52`

5. 禁用特定指令
    
    >rename-command CONFIG ""
    
    - 指定為空字串，即不允許執行該命令


# 參考資料
1. [Redis Security](https://redis.io/topics/security)