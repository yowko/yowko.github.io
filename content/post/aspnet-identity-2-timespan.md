---
title: "關於 ASP.NET Identity 2 時效設定"
date: 2017-12-18T02:44:00+08:00
lastmod: 2021-08-02T02:44:25+08:00
draft: false
tags: ["ASP.NET Identity"]
slug: "aspnet-identity-2-timespan"
aliases:
    - /2017/12/spnet-identity-2-timespan.html
---
## 關於 ASP.NET Identity 2 時效設定

之前筆記 [ASP.NET Identity 2 啟用自動鎖定機制](/aspnet-identity-2-lockout) 與 [改 ASP.NET Identity 2 的 Token 時效](/aspnet-identity-2-token-lifetime)，分別介紹到如何修改自動鎖定的結束時間以及產生 token 的預設時效，雖然兩者作用不同，但卻都有使用到一個 TimeSpan 設定，自己在這兩個設定上都遇到極為類似的問題：`無法設定時效永不過期`，所以趁著假日來紀錄一下可以怎麼做

## 時效設定機制

為了要正確設定，首先就是了解運作機制，避免自以為是的設定反而造成 bug

1. 自動鎖定的時效設定

    * [Source Code](https://github.com/aspnet/AspNetIdentity/blob/9c48993a446288032f9824633e6dae81257da06e/src/Microsoft.AspNet.Identity.Core/UserManager.cs)

    * 預設為 `TimeSpan.Zero`

        ![1lockoutdefault](https://user-images.githubusercontent.com/3851540/34082610-7f255a9c-e39c-11e7-9b95-ebed92d906c0.png)

    * 透過 `DefaultAccountLockoutTimeSpan` 來設定

        ![2setdafaultlockout](https://user-images.githubusercontent.com/3851540/34082611-7f51e3a0-e39c-11e7-9f54-aefc483d8094.png)

    * 時效使用機制

        > 達到設定的登入錯誤嘗試上限時，將 `現在的 UTC 時間` 加上 `DefaultAccountLockoutTimeSpan` 當作 `鎖定結束時間` 並寫入 db

2. Token 的時效設定

    * [Source Code](https://github.com/aspnet/AspNetIdentity/blob/9c48993a446288032f9824633e6dae81257da06e/src/Microsoft.AspNet.Identity.Owin/DataProtectorTokenProvider.cs)

    * 預設為 `TimeSpan.FromDays(1)`

        ![3defaulttokenlifespan](https://user-images.githubusercontent.com/3851540/34082612-7f8d7cee-e39c-11e7-885d-ffe95cb63d47.png)

    * 透過 `DataProtectorTokenProvider` 的 `TokenLifespan` 設定

        ![4settokelifespan](https://user-images.githubusercontent.com/3851540/34082613-7fc5d544-e39c-11e7-9f63-7f02e97f7468.png)

    * 時效使用機制

        > 產生 token 時僅將當下的 UTC 時間加入 hash，驗證時才加上 `TokenLifespan` 與驗證當下的時間進行比對

## 設定過期最大值

> 除了關閉過期檢查之外，沒有一個選項是可以直接指定永不過期的

1. 無法直接設定 `TimeSpan.MaxValue` (10675199.02:48:05.4775807)

    * 自動鎖定時就會出錯

        ![5timespanmax](https://user-images.githubusercontent.com/3851540/34082614-7ffd98a8-e39c-11e7-8a84-9838b5b83036.png)

    * Token 時效無法通過驗證

        ![6tokelifespan](https://user-images.githubusercontent.com/3851540/34082615-80357c32-e39c-11e7-9318-a0c8f90ce33a.png)

2. 無法直接設定 `TimeSpan.MaxValue` 原因

    * 超出 DateTime 最大值(DateTime.MaxValue： 9999/12/31 下午 11:59:59 )
    * 無論是自動鎖定還是 Token 時效，最終都會將 TimeSpan 加上一個 UTC 時間

3. 如何設定極大值

    * 理論值

        ```cs
        TimeSpan.FromTicks((DateTimeOffset.MaxValue-DateTimeOffset.UtcNow).Ticks)
        ```

        > 因為機制不同(產生 token 當下僅紀錄時間，加上 timespan 是發生在比對當下)，所以這個設定雖然無法完整精準滿足 `DateTimeOffset.MaxValue`(因為 TimeSpan 計算是發生在比對當下，而發 token 產生的時間點，因此不會出現 token 時間 + TimeSpan 超出 DateTime 上限的問題)

    * 實際值

        > 理論值可以取得 `DateTime.MaxValue` 與 `DateTimeOffset.UtcNow` 的精準落差，但實際上可用度很低，原因是該實際執行時會有時間差問題

        * 建議設定一個可接受的極大值即可(5000/12/31)

            ```cs
            TimeSpan.FromTicks(new DateTime(5000,12,31).Ticks)
            ```

        * 可以透過時間差調整設定找出最接近的極限值

            ```cs
            TimeSpan.FromTicks((DateTime.MaxValue - DateTime.UtcNow.AddSeconds(1)).Ticks)
            ```

## 心得

兩種機制應該是不同人(or team) 寫的，風格不太一致，以結果來看 Token 時效的強健性(robustness)較佳：不會第一時間就噴錯誤，但也較難 debug：不會提供明顯錯誤只說 token 無效

原本對 TimeSpan 機制並不是非常有把握，經過翻查 source code 以及做了些實驗後，比較清楚整個運作原理跟設計概念，使用上應該能更得心應手

## 參考資訊

1. [ASP.NET Identity 2 啟用自動鎖定機制](/aspnet-identity-2-lockout)
2. [改 ASP.NET Identity 2 的 Token 時效](/aspnet-identity-2-token-lifetime)
