---
title: "使用 MongoDB 儲存 ASP.NET Identity 資料"
date: 2018-07-20T16:36:00+08:00
lastmod: 2018-07-21T21:45:51+08:00
draft: false
tags: ["ASP.NET Identity","MongoDB"]
slug: "aspnet-identity-mongodb"
aliases:
    - /2018/07/aspnet-identity-mongodb.html
---
# 使用 MongoDB 儲存 ASP.NET Identity 資料
近期處理的某個專案需要 website 來呈現報表類資訊，起初 user 說只需要基本的登入，不需要其他使用者相關功能(e.g. 修改密碼、權限管理、帳號申請...etc)，於是便透過 [如何在 ASP.NET MVC 加上簡易表單驗證](https://blog.yowko.com/2017/03/aspnet-mvc.html) 中提到的小技巧快速地完成了一個有登入功能的網站，不過 user 在使用過幾次後還是覺得使用者相關功能 `nice to have`，但尷尬的事來了：user 一直覺得 website 有 "完整" 的使用者功能，只是未開放 "登入" 之外的其他功能，殊不知在建立 website 登入機制時根本連 DB 權限都沒申請，相關使用者 credential 資訊都存在 web.config 中

後來想到該 website 原本就有專屬的 MongoDB 來儲存相關 log，於是興起將 Identity 資料儲存在 MongoDB 的念頭，立馬來看看如何使用 MongoDB 搭配 ASP.NET Identity 

## 基本設定
1. 使用 ASP.NET MVC 專案範本
    
    ![1projecttemplate](https://user-images.githubusercontent.com/3851540/42991954-c32c88fa-8c39-11e8-85a2-d45cff9f1185.png)
2. 選擇 authentication type 為 Individual User Accounts
    
    ![2identity](https://user-images.githubusercontent.com/3851540/42991955-c36e401a-8c39-11e8-9183-1fccca8beb2b.png)

## 調整 NuGet packages
1. 移除 `Microsoft.AspNet.Identity.EntityFramework`
    
    ```
    Uninstall-Package Microsoft.AspNet.Identity.EntityFramework
    ```
    
    ![3removeidef](https://user-images.githubusercontent.com/3851540/42991956-c3967eb8-8c39-11e8-8a2f-f2ccc64f4c76.png)
2. 移除 `EntityFramework`
    
    ```
    Uninstall-Package EntityFramework
    ```
    
    ![4removeef](https://user-images.githubusercontent.com/3851540/42991957-c3c11998-8c39-11e8-89b2-d7198dff5da5.png)
3. 安裝 `AspNet.Identity.MongoDB`
    
    ```
    Install-Package AspNet.Identity.MongoDB
    ```
    
    ![5installaspidmongo](https://user-images.githubusercontent.com/3851540/42991958-c3e7cbce-8c39-11e8-82c6-49601ddaf4e5.png)

## 調整 namespace
1. IdentityModels.cs
    
    >預設路徑：`~/Models/IdentityModels.cs`
    
    - 移除 `using Microsoft.AspNet.Identity.EntityFramework;` 及 `using System.Data.Entity;`
    - 加入 `using AspNet.Identity.MongoDB;` 及 `using MongoDB.Driver;`
    - 修改 `ApplicationDbContext` class 
        
        ```cs
        public class ApplicationDbContext : IDisposable
        {
            public static ApplicationDbContext Create()
            {
                var client = new MongoClient("mongodb://127.0.0.1:27017");
                var database = client.GetDatabase("Identity");
                var users = database.GetCollection<ApplicationUser>("users");
                var roles = database.GetCollection<IdentityRole>("roles");
                return new ApplicationDbContext(users, roles);
            }
            private ApplicationDbContext(IMongoCollection<ApplicationUser> users, IMongoCollection<IdentityRole> roles)
            {
                Users = users;
                Roles = roles;
            }

            public IMongoCollection<IdentityRole> Roles { get; set; }

            public IMongoCollection<ApplicationUser> Users { get; set; }
            public void Dispose()
            {
            }
        }
        ```
    - 完整程式碼
        
        ```cs
        using System;
        using System.Configuration;
        using System.Security.Claims;
        using System.Threading.Tasks;
        using AspNet.Identity.MongoDB;
        using Microsoft.AspNet.Identity;
        using Microsoft.AspNet.Identity.Owin;
        using MongoDB.Driver;

        namespace IdentitySample.Models
        {
            // You can add profile data for the user by adding more properties to your ApplicationUser class, please visit https://go.microsoft.com/fwlink/?LinkID=317594 to learn more.
            public class ApplicationUser : IdentityUser
            {
                public async Task<ClaimsIdentity> GenerateUserIdentityAsync(UserManager<ApplicationUser> manager, string authenticationType)
                {
                    // Note the authenticationType must match the one defined in CookieAuthenticationOptions.AuthenticationType
                    var userIdentity = await manager.CreateIdentityAsync(this, authenticationType);
                    // Add custom user claims here
                    return userIdentity;
                }
            }

            public class ApplicationDbContext : IDisposable
            {
                public static ApplicationDbContext Create()
                {
                    var client = new MongoClient("mongodb://127.0.0.1:27017");
                    var database = client.GetDatabase("Identity");
                    var users = database.GetCollection<ApplicationUser>("users");
                    var roles = database.GetCollection<IdentityRole>("roles");
                    return new ApplicationDbContext(users, roles);
                }
                private ApplicationDbContext(IMongoCollection<ApplicationUser> users, IMongoCollection<IdentityRole> roles)
                {
                    Users = users;
                    Roles = roles;
                }

                public IMongoCollection<IdentityRole> Roles { get; set; }

                public IMongoCollection<ApplicationUser> Users { get; set; }
                public void Dispose()
                {
                }
            }
        }
        ``` 
3. IdentityConfig.cs
    
    > 預設位置：` ~/App_Start/IdentityConfig.cs`
    
    - 移除 `using Microsoft.AspNet.Identity.EntityFramework;` 及 `using System.Data.Entity;`
    - 加入 `using AspNet.Identity.MongoDB;`
    - 修改 `ApplicationUserManager` 的建立
        
        ```cs
        var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>().Users));
        ``` 
    - 完整程式碼
        
        ```cs
        using System;
        using System.Security.Claims;
        using System.Threading.Tasks;
        using AspNet.Identity.MongoDB;
        using Microsoft.AspNet.Identity;
        using Microsoft.AspNet.Identity.Owin;
        using Microsoft.Owin;
        using Microsoft.Owin.Security;
        using IdentitySample.Models;

        namespace IdentitySample
        {
            public class EmailService : IIdentityMessageService
            {
                public Task SendAsync(IdentityMessage message)
                {
                    // Plug in your email service here to send an email.
                    return Task.FromResult(0);
                }
            }

            public class SmsService : IIdentityMessageService
            {
                public Task SendAsync(IdentityMessage message)
                {
                    // Plug in your SMS service here to send a text message.
                    return Task.FromResult(0);
                }
            }

            // Configure the application user manager used in this application. UserManager is defined in ASP.NET Identity and is used by the application.
            public class ApplicationUserManager : UserManager<ApplicationUser>
            {
                public ApplicationUserManager(IUserStore<ApplicationUser> store)
                    : base(store)
                {
                }

                public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context) 
                {
                    var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>().Users));
                    // Configure validation logic for usernames
                    manager.UserValidator = new UserValidator<ApplicationUser>(manager)
                    {
                        AllowOnlyAlphanumericUserNames = false,
                        RequireUniqueEmail = true
                    };

                    // Configure validation logic for passwords
                    manager.PasswordValidator = new PasswordValidator
                    {
                        RequiredLength = 6,
                        RequireNonLetterOrDigit = true,
                        RequireDigit = true,
                        RequireLowercase = true,
                        RequireUppercase = true,
                    };

                    // Configure user lockout defaults
                    manager.UserLockoutEnabledByDefault = true;
                    manager.DefaultAccountLockoutTimeSpan = TimeSpan.FromMinutes(5);
                    manager.MaxFailedAccessAttemptsBeforeLockout = 5;

                    // Register two factor authentication providers. This application uses Phone and Emails as a step of receiving a code for verifying the user
                    // You can write your own provider and plug it in here.
                    manager.RegisterTwoFactorProvider("Phone Code", new PhoneNumberTokenProvider<ApplicationUser>
                    {
                        MessageFormat = "Your security code is {0}"
                    });
                    manager.RegisterTwoFactorProvider("Email Code", new EmailTokenProvider<ApplicationUser>
                    {
                        Subject = "Security Code",
                        BodyFormat = "Your security code is {0}"
                    });
                    manager.EmailService = new EmailService();
                    manager.SmsService = new SmsService();
                    var dataProtectionProvider = options.DataProtectionProvider;
                    if (dataProtectionProvider != null)
                    {
                        manager.UserTokenProvider = 
                            new DataProtectorTokenProvider<ApplicationUser>(dataProtectionProvider.Create("ASP.NET Identity"));
                    }
                    return manager;
                }
            }

            // Configure the application sign-in manager which is used in this application.
            public class ApplicationSignInManager : SignInManager<ApplicationUser, string>
            {
                public ApplicationSignInManager(ApplicationUserManager userManager, IAuthenticationManager authenticationManager)
                    : base(userManager, authenticationManager)
                {
                }

                public override Task<ClaimsIdentity> CreateUserIdentityAsync(ApplicationUser user)
                {
                    return user.GenerateUserIdentityAsync((ApplicationUserManager)UserManager);
                }

                public static ApplicationSignInManager Create(IdentityFactoryOptions<ApplicationSignInManager> options, IOwinContext context)
                {
                    return new ApplicationSignInManager(context.GetUserManager<ApplicationUserManager>(), context.Authentication);
                }
            }
        }
        ``` 
## 心得
雖然過去一直有印象可以將 ASP.NET Identity 的資料儲存至 MongoDB，但過去專案都還是透過 relation DB 來儲存資料，幸虧平常有留意相關應用讓這次臨時加入的需求不至影響到專案的時程

至於 ASP.NET Identity 與 MongoDB 的整合，在沒有 Microsoft 官方的支持，多虧有網路上的神人願意分享，雖然文件及使用的便利性仍有很大的改進空間，但不用自己從頭處理已是大幸呀

# 參考資訊
1. [如何在 ASP.NET MVC 加上簡易表單驗證](https://blog.yowko.com/2017/03/aspnet-mvc.html)
2. [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](https://blog.yowko.com/2017/11/add-aspnet-identity-empty-project.html)
3. [g0t4/aspnet-identity-mongo](https://github.com/g0t4/aspnet-identity-mongo/tree/AspNet.Identity.MongoDB)
4. [maxiomtech/MongoDB.AspNet.Identity](https://github.com/maxiomtech/MongoDB.AspNet.Identity)