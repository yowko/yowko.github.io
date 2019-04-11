---
title: "Windows Server 2016 docker 執行身份"
date: 2018-01-14T23:47:00+08:00
lastmod: 2018-10-02T23:47:45+08:00
draft: false
tags: ["Docker","Windows Server 2016"]
slug: "windows-server-2016-docker-users"
aliases:
    - /2018/01/windows-server-2016-docker-users.html
---
# Windows Server 2016 docker 執行身份
最近重新安裝幾台了電腦，一開始都沒有 docker 的執行權限，原因就是執行 docker for windows 需要有特殊權限：`docker-users`，而 `docker-users` 預設僅加入 `Administrator` 造成使用其他身份登入時都無法啟動 docker

今天又遇到，順手截個圖，紀錄一下

## Access Denied

![1accessdenied](https://user-images.githubusercontent.com/3851540/34917161-ada16540-f97d-11e7-9979-1f10ac70eeee.png)

## 調整使用者群組

*   方法 一：
    1.  Windows 系統管理工具

        ![2systemtool](https://user-images.githubusercontent.com/3851540/34917162-adcd15e6-f97d-11e7-9781-6300114ccfbe.png)

    2.  電腦管理

        ![3commanage](https://user-images.githubusercontent.com/3851540/34917163-adf9cf6e-f97d-11e7-908f-e1c076d67904.png)

    3.  本機使用者和群組 --> 群組 --> docker-users

        ![4dockerusers](https://user-images.githubusercontent.com/3851540/34917164-ae22c68a-f97d-11e7-8c90-2a15278ab197.png)

*   方法 二
    1.  Win+R 搜尋 `lusrmgr.msc`

        ![5lusrmgr](https://user-images.githubusercontent.com/3851540/34917165-ae4b0bd6-f97d-11e7-9d66-0cd5b16ee651.png)

    2.  群組 --> docker-users

        ![6dockerusers](https://user-images.githubusercontent.com/3851540/34917166-ae72f8da-f97d-11e7-9ace-fb2006f69c12.png)

*   docker-users 預設僅加入 `Administrator`

    ![7default](https://user-images.githubusercontent.com/3851540/34917167-ae9a5b64-f97d-11e7-83a0-233984191e11.png)

## 調整 docker-users 後需重新登入才會生效

![8relogin](https://user-images.githubusercontent.com/3851540/34917168-aeed44e6-f97d-11e7-80d7-364f4dc8d8f8.png)

## 心得

不知道為什麼 docker for windows 要另外建立一個自己的群組，原因不明、功能不明、實際作用不明，雖然只是一個小小設定但在沒有完整 windows 管理權限的環境中還是很麻煩的

# 參考資訊

1.  [Windows 7: Local Users and Groups Manager - Open](https://www.sevenforums.com/tutorials/7539-local-users-groups-manager-open.html)
