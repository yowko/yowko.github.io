---
title: "ASP.NET Identity 修改預設 Cookie 名稱"
date: 2018-01-13T15:09:00+08:00
lastmod: 2018-10-02T15:09:40+08:00
draft: false
tags: ["ASP.NET Identity"]
slug: "aspnet-identity-change-cookiename"
aliases:
    - /2018/01/aspnet-identity-change-cookiename.html
---
# ASP.NET Identity 修改預設 Cookie 名稱
最近專案同時有前台與後台在進行開發，過去習慣的做法是將前後台使用同一個 VS 專案 (project) 進行開發，再透過 ASP.NET MVC Area 的功能來切分，但因為這個專案的前後台會分別部署在不同的 server 中所以就拆成兩個 vs project 但仍在同一個 vs 方案 (solution) 中，結果在開發時出現同時開啟前後台時會讓 cookie 混淆(前後台會吃到對方的 user 資訊)，這讓前後台的同時開發變得產生出現靈異 bug

今天就來紀錄該如何透過設定 cookie name 的方式來避免前後台 cookie 互咬

## 實際情況

1.  預設使用 `.AspNet.ApplicationCookie` 來保存登入狀態
2.  前後台帳號無法共用
    *   後台使用 `test@test.com`

        ![1bo](https://user-images.githubusercontent.com/3851540/34903692-fe1547ba-f871-11e7-928a-f73d05cb0e1c.png)

    *   前台使用 `yowko@yowko.com`

        ![2portal](https://user-images.githubusercontent.com/3851540/34903693-fe3e420a-f871-11e7-851a-1c24396079b8.png)

3.  前後台共用登入資訊

    > 共用 `.AspNet.ApplicationCookie` 內容

    *   後台

        ![3boshare](https://user-images.githubusercontent.com/3851540/34903694-fe664516-f871-11e7-9026-843a4c2e6c6c.png)

    *   後台

        ![4portalshare](https://user-images.githubusercontent.com/3851540/34903695-fe8e0e34-f871-11e7-88da-cca9d0dab316.png)

## 修改 cookie name

1.  `App_Start` --> `Startup.Auth.cs`

    ![5StartupAuth](https://user-images.githubusercontent.com/3851540/34903696-feb44144-f871-11e7-8bb5-40049c8cc824.png)

2.  ConfigureAuth 方法中，在  app.UseCookieAuthentication 加入 `CookieName`

    ```cs
    app.UseCookieAuthentication(new CookieAuthenticationOptions
    {
        AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
        LoginPath = new PathString("/Account/Login"),
        Provider = new CookieAuthenticationProvider
        {
            // Enables the application to validate the security stamp when the user logs in.
            // This is a security feature which is used when you change a password or add an external login to your account.  
            OnValidateIdentity = SecurityStampValidator.OnValidateIdentity<ApplicationUserManager, ApplicationUser>(
                validateInterval: TimeSpan.FromMinutes(30),
                regenerateIdentity: (manager, user) => user.GenerateUserIdentityAsync(manager))
        },
        CookieName = "backoffice"
    });
    ```

## 實際效果

> 前後台各使用指定的 cookie name 來紀錄登入資訊未再互相影響

*   後台已正確使用 `test@test.com`

    ![6bo](https://user-images.githubusercontent.com/3851540/34903697-fedbb026-f871-11e7-9f63-8b22eba335b6.png)

*   前台已正確使用 `yowko@yowko.com`

    ![7portal](https://user-images.githubusercontent.com/3851540/34903698-ff030e32-f871-11e7-900f-32e7ac92968c.png)

## 心得

cookie 互相汙染的狀況在不同的 domain 下是不會發生的，只有開發環境或是測試環境可能因為資源較不足，不一定有足夠的 domain 供測試使用才會出現，但既然遇到問題當然就是藉機研究一番解決問題囉

# 參考資訊

1.  [Rename Authentication Cookie Name of Asp.Net Identity](http://tech.trailmax.info/2014/07/rename-authentication-cookie-name-of-asp-net-identity/)
