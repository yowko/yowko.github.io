---
title: "ASP.NET Identity 2 啟用自動鎖定機制"
date: 2017-12-11T22:31:00+08:00
lastmod: 2021-11-03T22:31:05+08:00
draft: false
tags: ["ASP.NET Identity"]
slug: "aspnet-identity-2-lockout"
aliases:
    - /2017/12/aspnet-identity-2-lockout.html
---
## ASP.NET Identity 2 啟用自動鎖定機制

新專案有個需求是要讓 user 在嘗試登入時超過指定登入次數就自動鎖定帳號讓 user 無法登入，而 ASP.NET Identity 本身就已經涵蓋了這個功能，不需要自己刻，在專案開始前的技術評估階段讓 ASP.NET Identity 加了一點分數。

在實際使用時花了一些時間才真正完成設定，主要當然是不清楚其中運作機制所造成的。後來在協助同事引用相關機制時，發現自己還不是很清楚過程跟設定(無法做正確及系統化的說明)，所以趁著這個機會做個紀錄好好釐清一下

## 確認啟用 user 的 LockoutEnabled 設定

1. 預設啟用
    * 透過設定 ApplicationUserManager 的 UserLockoutEnabledByDefault 屬性

        > 將會在建立 user 時啟用自動鎖定

    * 依 ASP.NET Identity 2 預設範本為例

        * App_Start --> IdentityConfig.cs

            ![1identityconfig](https://user-images.githubusercontent.com/3851540/33835121-7041d59a-dec0-11e7-8e98-7fc85a3abd5d.png)

        * ApplicationUserManager --> Create

            ```cs
            manager.UserLockoutEnabledByDefault = true;
            ```

            ![2defaultenable](https://user-images.githubusercontent.com/3851540/33835122-706b07da-dec0-11e7-8fb0-01d2d1d8bd69.png)

2. 手動啟用
    * 依實際需求調整 user 是否啟用自動鎖定
        * ApplicationUserManager.SetLockoutEnabled(userId,bool)
        * ApplicationUserManager.SetLockoutEnabledAsync(userId,bool)

## 啟用 user 登入失敗紀錄機制

* 這是在執行登入動作上的參數設定

    > 可以依需求來決定該次登入動作是否會被紀錄

* 預設不啟用 (所有登入行為皆不會累計失敗紀錄)
* 依 ASP.NET Identity 2 預設範本為例
  * Controllers --> AccountController.cs

    ![4accountcontroller](https://user-images.githubusercontent.com/3851540/33835117-6fc4bc90-dec0-11e7-8364-038206bd03dc.png)

  * AccountController --> Login

    > shouldLockout 改為 true

    ```cs
    var result = await SignInManager.PasswordSignInAsync(model.Email, model.Password, model.RememberMe, shouldLockout: true);
    ```

    ![4shouldlock](https://user-images.githubusercontent.com/3851540/33835118-6fecf912-dec0-11e7-9028-aad60b4f1165.png)

## 自動鎖定機制說明

1. 登入行為是否列入錯誤紀錄

    > 依 SignInManager.PasswordSignInAsync 方法的 shouldLockout 參數設定，未啟用將不會累計錯誤次數

    ```cs
    SignInManager.PasswordSignInAsync(model.Email, model.Password, model.RememberMe, shouldLockout: true);
    ```

2. 檢核是否鎖定
    * 由儲存 user 資料的三個 table 欄位控制
        * LockoutEnabled

            > 是否啟用自動鎖定

        * LockoutEndUtc

            > 鎖定結束時間

        * AccessFailedCount

            > 登入失敗次數

    * 實際驗證是否鎖定流程
        * LockoutEnabled：`True` 且時間小於 `LockoutEndUtc` 即鎖定

            ![3lockout](https://user-images.githubusercontent.com/3851540/33835116-6f9ace08-dec0-11e7-90b5-92a39ad661fe.png)

        * LockoutEnabled：`True` 且時間 `不` 小於 `LockoutEndUtc` 可以執行登入
            * 登入失敗次數達指定數目 (AccessFailedCount) 依 DefaultAccountLockoutTimeSpan 設定鎖定結束時間

        * LockoutEnabled：`False` 可以執行登入，沒有鎖定機制
            * 登入失敗次數達指定數目 (AccessFailedCount) 仍會依 DefaultAccountLockoutTimeSpan 設定鎖定結束時間，但`不會鎖定登入`

    * 結論：
        * 同時滿足啟用自動鎖定 (LockoutEnabled 為 true) 且當下時間小於鎖定結束時間 (LockoutEndUtc)，即 `鎖定，無法登入`
        * 未啟用自動鎖定 (LockoutEnabled 為 false)，登入嘗試紀錄及寫入鎖定時間的機制依舊，但因未滿足上述條件(LockoutEnabled 為 true)，所以不會鎖定

## ApplicationUserManager 的自動鎖定預設設定

![5defaultset](https://user-images.githubusercontent.com/3851540/33835120-70180ec2-dec0-11e7-8e07-a0bdc133352f.png)

1. UserLockoutEnabledByDefault

    > 將會在建立 user 時啟用自動鎖定

2. DefaultAccountLockoutTimeSpan

    > 鎖定的預設時間間隔，達到登入失敗上限時間點開始鎖定的時間間隔，預設為五分鐘(達到失敗上限後鎖定五分鐘不得登入)

3. MaxFailedAccessAttemptsBeforeLockout

    > 登入失敗次數上限

## 心得

平心而論機制並不算複雜，各個設定也有明確的相關說明，只是散落在各處，如果沒有將各處的程式及設定看過一遍其實很難一次設定正確，也造成無形中的使用進入門檻

透過這次整理，讓我對整個流程跟機制清晰很多，ASP.NET Identity 2 相關設定相當簡潔也很好理解滿只是說明還是有改善空間，不然就白白浪費了一個好用的工具

另外需要特別注意的是鎖定時間使用的是 `UTC 時間`，這個在測試時一不小心就忽略了

## 參考資訊

1. [OAuth Server 鎖定(Lockout)登入失敗次數太多的帳號](https://dotblogs.com.tw/yc421206/2016/08/03/asp_net_identity_oauth_user_lockout)
