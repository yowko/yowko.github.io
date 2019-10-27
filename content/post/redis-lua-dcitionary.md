---
title: "在 Redis 中使用 Lua 的 Dictionary"
date: 2019-10-26T21:30:00+08:00
lastmod: 2016-10-26T21:30:31+08:00
draft: false
tags: ["Redis","Lua"]
slug: "redis-lua-dcitionary"
---

## 在 Redis 中使用 Lua 的 Dictionary

最近幾周一直在趕專案進度，也累積了好幾篇筆記沒有動手，想著如果再不紀錄可能就想不起來當時遇到問題的情境了 ~~~

最近專案主要都是使用 Lua 來存取 Redis，對於過去多數使用 c# 套件的我而言就是東撞個牆西踩個雷的，於是趕緊趁著專案空檔紀錄一下個人使用經驗(方便~~專案有得抄~~日後參考？)

雖然在 Lua 之中並沒有 Dictionary，不過還是可以透過 `table` 來模擬，立馬來看看可以怎麼做吧

## 基本環境說明

1. macOS Majave 10.14.6
2. Redis 5.0.6

    > 使用 docker 啟動 redis instance

    ```bash
    docker run -d -p 6379:6379 redis
    ```

3. ZeroBrane Studio 1.80

## 情境說明

1. 使用 `sets` 儲存所有 userId

    ```bash
    SADD users 1 3 5
    ```

2. 使用 `hashes` 儲存 user 資料

    ```bash
    HMSET userdata:1 name "yowko" email "yowko@yowko.com"
    HMSET userdata:3 name "test" email "test@test.com"
    HMSET userdata:5 name "poc" email "poc@poc.com"
    ```

3. 測試用資料結構

    ![1datastructure](https://user-images.githubusercontent.com/3851540/67637098-16472680-f912-11e9-8922-c8579a97d1f2.png)

## Lua

```lua
local allUsersKey="users"
local userdataPrefixKey="userdata"
-- 取得所有 user
local allUsers = redis.call("smembers",  allUsersKey)

--逐一處理所有 user
local results = {} for userIndex, userId in ipairs(allUsers) do
  -- 單一 user 的 key
  local userdataKey=userdataPrefixKey .. ":" .. userId
  -- 取得單一 user 的所有屬性值
  local getAll= redis.call("HGETALL", userdataKey)
  -- 暫存欄位名稱
  local tmpSubKey=""
  -- 處理所有欄位屬性
  local tmpSubData = {} for index,value in ipairs (getAll) do
    -- 如果 index 是單數則為欄位名稱，偶數為屬性值
    if index % 2 == 1 then
      tmpSubKey= value
    else
      -- 使用名稱設定 table 
      tmpSubData[tmpSubKey]=value
    end
  end
  -- 將個別 user 的所有屬性存入 table 中
  results[userId]=tmpSubData
end

return results
```

## 心得

雖然透過上面的 lua script 可以模擬出 dictionary 的樣子，但實際使用上還是有限制，其中最大的問題就是無法直接輸出 table (以下列出差異)

- 一般 table (正常輸出)

    > table.insert(results,userId)

    ![2normaltable](https://user-images.githubusercontent.com/3851540/67637099-16dfbd00-f912-11e9-96bf-493702801b02.png)

- 使用 Dictionary (未輸內容)

    > results[userId]=tmpSubData

    ![3dictionary](https://user-images.githubusercontent.com/3851540/67637100-16dfbd00-f912-11e9-98a3-575588601154.png)

至於該如何解決，請待下回筆記分享個人(不知道是不是正確)的做法

## 參考資訊

1. [[Lua] Table使用手冊](http://huli.logdown.com/posts/198866-lua-table)
2. [[Lua] 程式設計教學：使用表 (table)](https://michaelchen.tech/lua-programming/table/)
