---
title: "清除 Windows 上的 git 驗證資訊(Credential)"
date: 2018-04-17T21:00:00+08:00
lastmod: 2018-10-04T21:00:43+08:00
draft: false
tags: ["Git","Windows 10"]
slug: "clear-windows-credential"
aliases:
    - /2018/04/clear-windows-credential.html
---
# 清除 Windows 上的 git 驗證資訊(Credential)
公司電腦的安全性設定要求固定時間需要修改個人 AD 密碼，連帶其他內部系統密碼也會一並被修改，之前修改 AD 密碼後，密碼驗證失敗會彈出錯誤訊息要求重新輸入，但最近卻出現吐出驗證失敗沒有要求重新輸入，造成與 git server 的交互動作全部失敗，順手紀錄一下個人做法：將 Windows 上紀錄的 git credentials 清除，需要與 git server 溝通時重新輸入

## 紀錄 Credential

> 以下主要使用 Windows 10 為例

1.  Clone git repository 時會出現 Windows Security 視窗要求填入帳號密碼

    > ![1gitclone](https://user-images.githubusercontent.com/3851540/38846321-6a7b932c-422e-11e8-9203-c76282650ab2.png)

    > ![2winsecurity](https://user-images.githubusercontent.com/3851540/38846323-6aa4217a-422e-11e8-9f23-ab99b6da35ac.png)

2.  取消 Windows Security 視窗後會改跳 git CLI stdin wrapper 視窗要求帳號密

    > ![3gitcli](https://user-images.githubusercontent.com/3851540/38846324-6ace243e-422e-11e8-8897-fd0f3205bf7f.png)

3.  兩者皆會將帳號密碼寫入 Windows Credential 中

    > 我個人經驗 Windows 7 上的 git CLI stdin wrapper 不會寫入

## 如何清除 git

1.  開啟 Windows Credentials

    > 下列方法擇一即可

    *   直接搜尋 `Manage Windows Credentials`

        > ![4search](https://user-images.githubusercontent.com/3851540/38846326-6af7da90-422e-11e8-8487-693f4c0d2d45.png)

    *   Control Panel --> User Accounts --> Manage Windows Credentials

        > ![5controlpanel](https://user-images.githubusercontent.com/3851540/38846327-6b224ece-422e-11e8-818b-802d2c07fdcc.png)

        > ![6coountmanage](https://user-images.githubusercontent.com/3851540/38846328-6b4a5fc2-422e-11e8-95f3-66db537c4364.png)

2.  針對想要移除的 credential 按下 `Remove`

    > ![7remove](https://user-images.githubusercontent.com/3851540/38846329-6b7675da-422e-11e8-95d9-8d907f627235.png)

    > ![8confirm](https://user-images.githubusercontent.com/3851540/38846330-6ba03280-422e-11e8-8c3a-b80e9b4bd2fa.png)

## 心得

不知道跟 git 版本有沒有關係，或者是 Windows 版本造成的，經驗中發生的次數並不多但也不算是罕見，偶爾會聽到其他同事反應，不過倒是沒有特定共同點，唯一相同的就是剛改過 AD 密碼，暫時還不確定真正的問題發生原因，就先紀錄備查吧

# 參考資訊
