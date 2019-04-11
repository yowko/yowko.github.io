---
title: "改 ASP.NET Identity 2 的 Token 時效"
date: 2017-12-15T00:37:00+08:00
lastmod: 2018-09-30T00:37:27+08:00
draft: false
tags: ["ASP.NET Identity"]
slug: "aspnet-identity-2-token-lifetime"
aliases:
    - /2017/12/aspnet-identity-2-token-lifetime.html
---
# 改 ASP.NET Identity 2 的 Token 時效
專案對於系統發出給 user 的專用連結(重設密碼、promotion link)有個共通的要求：需要可以指定過期時間。為連結加入時效限制可以加強資訊安全的管控，做法就是將時間 hash 後加入連結中，並在執行連結內容時驗證時間 hash 的有效性，概念並不複雜只是秉持著一貫的理念：站在巨人的肩膀上才能看得更遠。既然 ASP.NET Identity 已經提供的功能就不要自己刻，避免製造出隱藏的難解 bug

雖然 ASP.NET Identity 2 提供了基本功能，重點還是要知道如何應用才能發揮實際效果，今天就來看看可以如何設定 Token 過期時間

## 預設過期時間

*   **`一天`**
*   [source code](https://github.com/aspnet/AspNetIdentity/blob/9c48993a446288032f9824633e6dae81257da06e/src/Microsoft.AspNet.Identity.Owin/DataProtectorTokenProvider.cs)

    ```cs
    /// <summary>
    ///     Token provider that uses an IDataProtector to generate encrypted tokens based off of the security stamp
    /// </summary>
    public class DataProtectorTokenProvider<TUser, TKey> : IUserTokenProvider<TUser, TKey>
        where TUser : class, IUser<TKey> where TKey : IEquatable<TKey>
    {
        /// <summary>
        ///     Constructor
        /// </summary>
        /// <param name="protector"></param>
        public DataProtectorTokenProvider(IDataProtector protector)
        {
            if (protector == null)
            {
                throw new ArgumentNullException("protector");
            }
            Protector = protector;
            TokenLifespan = TimeSpan.FromDays(1);
        }
    ....
    ```

    ![1oneday](https://user-images.githubusercontent.com/3851540/34002943-f3228c8c-e12e-11e7-9343-2ea3abff77d4.png)

## 修改方式

> 預 ASP.NET MVC 的 Identity 預設範本為例

1.  開啟 App_Start --> IdentityConfig.cs

    ![2identityconfig](https://user-images.githubusercontent.com/3851540/34002944-f351739e-e12e-11e7-950a-3131a4b97c38.png)

2.  修改 ApplicationUserManager class 的 Create 方法：指定 DataProtectionProvider 的 TokenLifespan
    *   修改前

        ![3beforechange](https://user-images.githubusercontent.com/3851540/34002946-f37d43b6-e12e-11e7-851d-c080af6e89c2.png)

    *   修改後：指定 Token 時效為 `一個小時`

        ```cs
        if (dataProtectionProvider != null)
        {
            manager.UserTokenProvider =
                new DataProtectorTokenProvider<ApplicationUser>(
                    dataProtectionProvider.Create("ASP.NET Identity"))
                {
                    TokenLifespan = TimeSpan.FromHours(1)
                };
        }
        ```

        ![4after](https://user-images.githubusercontent.com/3851540/34002947-f3a89cdc-e12e-11e7-8b1e-d1ae55b2654c.png)

## 實際用法

> 預 ASP.NET MVC 的 Identity 預設範本為例

1.  Controllers --> AccountController.cs

    ![5accountcontroller](https://user-images.githubusercontent.com/3851540/34002949-f3d5674e-e12e-11e7-9341-067731370b81.png)

2.  ForgotPassword 中有使用方式介紹
    *   預設未啟用

        ![6disable](https://user-images.githubusercontent.com/3851540/34002950-f3ffa93c-e12e-11e7-9862-e14dce903ba9.png)

    *   使用說明

        ```cs
        //依 user id 產生專屬 token
        string code = await UserManager.GeneratePasswordResetTokenAsync(user.Id);
        //使用 user id 及 token 產生 reset password 的連結
        var callbackUrl = Url.Action("ResetPassword", "Account", 
            new { userId = user.Id, code = code }, protocol: Request.Url.Scheme); 
        // 將 reset password email 給 user
        await UserManager.SendEmailAsync(user.Id, 
            "Reset Password", 
            "Please reset your password by clicking <a href=\"" + callbackUrl + "\">here</a>");
        ```

        ![7enable](https://user-images.githubusercontent.com/3851540/34002951-f42decca-e12e-11e7-8675-25d033135eb8.png)

3.  實際收到的 mail 內容

    ![8actual](https://user-images.githubusercontent.com/3851540/34002953-f45a72d6-e12e-11e7-9f82-bf17b2c9624a.png)

## 心得

這個 Token 時效設定是個很小、很簡易的設定，只是文件上還是不太足夠，我一直找不到相關官方文獻說明預設值，後來是去翻 source code 才確認是 `一天`，不過至少官方文件有介紹如何修改設定，還不賴

透過簡單的設定過程就讓對外連結加入時效驗證，相當方便

# 參考資訊

1.  [Email Token Expiring After 15 mins - Asp Identity 2.0 API](https://stackoverflow.com/questions/27152612/email-token-expiring-after-15-mins-asp-identity-2-0-api)
2.  [ccount Confirmation and Password Recovery with ASP.NET Identity (C#)](https://docs.microsoft.com/en-us/aspnet/identity/overview/features-api/account-confirmation-and-password-recovery-with-aspnet-identity)
