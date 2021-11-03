---
title: "將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中 - MongoDB 版"
date: 2018-07-30T23:03:00+08:00
lastmod: 2021-11-03T23:03:39+08:00
draft: false
tags: ["ASP.NET Identity","ASP.NET MVC","MongoDB"]
slug: "aspnet-identity-aspnet-mvc-empty-mongodb"
aliases:
    - /2018/07/aspnet-identity-aspnet-mvc-empty-mongodb.html
---
## 將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中 - MongoDB 版

在之前筆記 [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/2017/11/add-aspnet-identity-empty-project.html) 中有提到我個人事實上比較喜歡 MVC Empty 專案範本的乾淨風格，當時也是因為不想讓專案中有過多預設套件及程式碼特別紀錄如何從 Empty 專案範本來逐步加入 ASP.NET Identity，最近專案剛好有用到 MongoDB 來儲存 ASP.NET Identity 資料 (詳細內容請參考 [使用 MongoDB 儲存 ASP.NET Identity 資料](/2018/07/aspnet-identity-mongodb.html))，所以順手紀錄一下如何逐步將 ASP.NET Identity 搭配 MongoDB 加至 ASP.NET MVC Empty 專案中

## 使用空專案範本建立專案

![EMPTY](https://user-images.githubusercontent.com/3851540/32415921-3c31f038-c27c-11e7-9392-e6f1e7d2ac94.png)

## 安裝 ASP.NET Identity 相關套件

1. Microsoft.AspNet.Identity.Owin
    - 相依套件
        - Microsoft.Owin.Security (>= 3.0.1)
        - Microsoft.AspNet.Identity.Core (>= 2.2.2)
        - Microsoft.Owin.Security.Cookies (>= 3.0.1)
        - Microsoft.Owin.Security.OAuth (>= 3.0.1)
    - 指令安裝

        ```cmd
        Install-Package Microsoft.AspNet.Identity.Owin
        ```

2. Install-Package AspNet.Identity.MongoDB
    - 相依套件
        - Microsoft.AspNet.Identity.Core (>= 2.2.1)
        - MongoDB.Driver (>= 2.0.0)
    - 指令安裝

        ```cmd
        Install-Package AspNet.Identity.MongoDB
        ```  

3. Microsoft.Owin.Host.SystemWeb
    - 相依套件
        - Owin (>= 1.0.0)
        - Microsoft.Owin (>= 4.0.0)
    - 指令安裝

        ```cmd
        Install-Package Microsoft.Owin.Host.SystemWeb
        ```  

## 在 Model 資料夾中加入 User 相關 model

1. 建立 `ApplicationUser` 並繼承 `IdentityUser`

    ```cs
    public class ApplicationUser: IdentityUser
    ```

2. 在 ApplicationUser 中建立 `GenerateUserIdentityAsync` 方法

    ```cs
    public async Task<ClaimsIdentity> GenerateUserIdentityAsync(UserManager<ApplicationUser> manager)
    {
        var userIdentity = await manager.CreateIdentityAsync(this, DefaultAuthenticationTypes.ApplicationCookie);
        return userIdentity;
    }
    ```

3. 完整程式碼

    ```cs
    using AspNet.Identity.MongoDB;
    using Microsoft.AspNet.Identity;
    using System.Security.Claims;
    using System.Threading.Tasks;

    namespace IdentityMongo.Models
    {
        public class ApplicationUser : IdentityUser
        {
            public async Task<ClaimsIdentity> GenerateUserIdentityAsync(UserManager<ApplicationUser> manager)
            {
                var userIdentity = await manager.CreateIdentityAsync(this, DefaultAuthenticationTypes.ApplicationCookie);
                return userIdentity;
            }
        }
    }
    ```

## 建立 MongoDB 連線物件

> 在 `Models` 資料夾中加入 `ApplicationDbContext.cs`

1. 建立 `ApplicationDbContext` 繼承至 `IDisposable`

    ```cs
    public class ApplicationDbContext : IDisposable
    {
    }
    ```

2. 實作 `Dispose()`

    ```cs
    public void Dispose()
    {
    }
    ```

3. 加入 `Users` 與 `Roles` 屬性

    ```cs
    public IMongoCollection<IdentityRole> Roles { get; set; }

    public IMongoCollection<ApplicationUser> Users { get; set; }
    ```

4. 建立 private 建構子並接受 users 與 roles 參數

    ```cs
    private ApplicationDbContext(IMongoCollection<ApplicationUser> users, IMongoCollection<IdentityRole> roles)
    {
        Users = users;
        Roles = roles;
    }
    ```

5. 建立 `static` 的 `Create()` 方法並回傳 `ApplicationDbContext`

    ```cs
    public static ApplicationDbContext Create()
    {
        //使用 mongo連線資訊建立 mongodb client
        var client = new MongoClient("mongodb://127.0.0.1:27017");
        //指定儲存的 db
        var database = client.GetDatabase("Identity");
        //指定 user 存放的 collection
        var users = database.GetCollection<ApplicationUser>("users");
        //指定 role 存放的 collection
        var roles = database.GetCollection<IdentityRole>("roles");
        // 回傳 private 建構子內容
        return new ApplicationDbContext(users, roles);
    }
    ```

6. 完整程式碼

    ```cs
    using AspNet.Identity.MongoDB;
    using MongoDB.Driver;
    using System;

    namespace IdentityMongo.Models
    {
        public class ApplicationDbContext : IDisposable
        {
            public static ApplicationDbContext Create()
            {
                //使用 mongo連線資訊建立 mongodb client
                var client = new MongoClient("mongodb://127.0.0.1:27017");
                //指定儲存的 db
                var database = client.GetDatabase("Identity");
                //指定 user 存放的 collection
                var users = database.GetCollection<ApplicationUser>("users");
                //指定 role 存放的 collection
                var roles = database.GetCollection<IdentityRole>("roles");
                // 回傳 private 建構子內容
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

## 建立 user 管理用物件

> 在 `App_Start` 資料夾中加入 `IdentityConfig.cs`

1. 加入 Email service

    ```cs
    public class EmailService : IIdentityMessageService
    {
        public Task SendAsync(IdentityMessage message)
        {
            // Plug in your email service here to send an email.
            return Task.FromResult(0);
        }
    }
    ```

2. 加入 SMS service

    ```cs
    public class SmsService : IIdentityMessageService
    {
        public Task SendAsync(IdentityMessage message)
        {
            // Plug in your SMS service here to send a text message.
            return Task.FromResult(0);
        }
    }
    ```

3. 建立 `ApplicationUserManager` class
    - 建立 `ApplicationUserManager` 繼承 `UserManager<ApplicationUser>`

        ```cs
        public class ApplicationUserManager : UserManager<ApplicationUser>
        {
        }
        ```

    - 加入使用 `IUserStore<ApplicationUser>` 為參數的建構子呼叫 base class 建構子

        ```cs
        public ApplicationUserManager(IUserStore<ApplicationUser> store) : base(store)
        {
        }
        ```

    - 建立 Create 方法

        ```cs
        public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context)
        {
            var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>().Users));
            // 設定 usernames 驗證邏輯
            manager.UserValidator = new UserValidator<ApplicationUser>(manager)
            {
                AllowOnlyAlphanumericUserNames = false,
                RequireUniqueEmail = true
            };
            // 設定密碼驗證邏輯
            manager.PasswordValidator = new PasswordValidator
            {
                RequiredLength = 6,
                RequireNonLetterOrDigit = true,
                RequireDigit = true,
                RequireLowercase = true,
                RequireUppercase = true,
            };
            // 設定 user 預設鎖定
            manager.UserLockoutEnabledByDefault = false;
            manager.DefaultAccountLockoutTimeSpan = TimeSpan.FromMinutes(5);
            manager.MaxFailedAccessAttemptsBeforeLockout = 5;
            // 設定兩因子驗證 (two factor authentication). 這邊示範使用簡訊及 Emails
            manager.RegisterTwoFactorProvider("Phone Code", new PhoneNumberTokenProvider<ApplicationUser>
            {
                MessageFormat = "Your security code is {0}"
            });
            manager.RegisterTwoFactorProvider("Email Code", new EmailTokenProvider<ApplicationUser>
            {
                Subject = "Security Code",
                BodyFormat = "Your security code is {0}"
            });
            // 設定 Email 服務
            manager.EmailService = new EmailService();
            // 設定簡訊服務
            manager.SmsService = new SmsService();
            var dataProtectionProvider = options.DataProtectionProvider;
            if (dataProtectionProvider != null)
            {
                manager.UserTokenProvider =
                    new DataProtectorTokenProvider<ApplicationUser>(dataProtectionProvider.Create("ASP.NET Identity"));
            }
            return manager;
        }
        ```

4. 建立 `ApplicationSignInManager` class

    ```cs
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
    ```

5. 完整程式碼

    ```cs
    using AspNet.Identity.MongoDB;
    using IdentityMongo.Models;
    using Microsoft.AspNet.Identity;
    using Microsoft.AspNet.Identity.Owin;
    using Microsoft.Owin;
    using Microsoft.Owin.Security;
    using System;
    using System.Security.Claims;
    using System.Threading.Tasks;

    namespace IdentityMongo
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
        public class ApplicationUserManager : UserManager<ApplicationUser>
        {
            public ApplicationUserManager(IUserStore<ApplicationUser> store) : base(store)
            {
            }
            public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context)
            {
                var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>().Users));
                // 設定 usernames 驗證邏輯
                manager.UserValidator = new UserValidator<ApplicationUser>(manager)
                {
                    AllowOnlyAlphanumericUserNames = false,
                    RequireUniqueEmail = true
                };

                // 設定密碼驗證邏輯
                manager.PasswordValidator = new PasswordValidator
                {
                    RequiredLength = 6,
                    RequireNonLetterOrDigit = true,
                    RequireDigit = true,
                    RequireLowercase = true,
                    RequireUppercase = true,
                };

                // 設定 user 預設鎖定
                manager.UserLockoutEnabledByDefault = false;
                manager.DefaultAccountLockoutTimeSpan = TimeSpan.FromMinutes(5);
                manager.MaxFailedAccessAttemptsBeforeLockout = 5;

                // 設定兩因子驗證 (two factor authentication). 這邊示範使用簡訊及 Emails
                manager.RegisterTwoFactorProvider("Phone Code", new PhoneNumberTokenProvider<ApplicationUser>
                {
                    MessageFormat = "Your security code is {0}"
                });
                manager.RegisterTwoFactorProvider("Email Code", new EmailTokenProvider<ApplicationUser>
                {
                    Subject = "Security Code",
                    BodyFormat = "Your security code is {0}"
                });
                // 設定 Email 服務
                manager.EmailService = new EmailService();
                // 設定簡訊服務
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

## 註冊 Identity 至 OWIN 中

1. 在 `App_Start` 目錄中加入 `Startup.Auth.cs`
    - 建立 partial `Startup` class

        ```cs
        public partial class Startup
        {
        }
        ```

    - 建立 `ConfigureAuth` 方法

        ```cs
        public void ConfigureAuth(IAppBuilder app)
        {
            app.CreatePerOwinContext(ApplicationDbContext.Create);
            app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
            app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);

            app.UseCookieAuthentication(new CookieAuthenticationOptions
            {
                AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
                LoginPath = new PathString("/Account/Register"),
                Provider = new CookieAuthenticationProvider
                {
                    OnValidateIdentity = SecurityStampValidator.OnValidateIdentity<ApplicationUserManager, ApplicationUser>(
                        validateInterval: TimeSpan.FromMinutes(30),
                        regenerateIdentity: (manager, user) => user.GenerateUserIdentityAsync(manager))
                }
            });
            app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);
        }
        ```

    - 完整程式碼

        ```cs
        using IdentityMongo.Models;
        using Microsoft.AspNet.Identity;
        using Microsoft.AspNet.Identity.Owin;
        using Microsoft.Owin;
        using Microsoft.Owin.Security.Cookies;
        using Owin;
        using System;

        namespace IdentityMongo
        {
            public partial class Startup
            {
                public void ConfigureAuth(IAppBuilder app)
                {
                    app.CreatePerOwinContext(ApplicationDbContext.Create);
                    app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
                    app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);

                    app.UseCookieAuthentication(new CookieAuthenticationOptions
                    {
                        AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
                        LoginPath = new PathString("/Account/Register"),
                        Provider = new CookieAuthenticationProvider
                        {
                            OnValidateIdentity = SecurityStampValidator.OnValidateIdentity<ApplicationUserManager, ApplicationUser>(
                                validateInterval: TimeSpan.FromMinutes(30),
                                regenerateIdentity: (manager, user) => user.GenerateUserIdentityAsync(manager))
                        }
                    });
                    app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);
                }
            }
        }
        ```

2. 在根目錄加入 `Startup.cs`
    - 建立 partial Startup class

        ```cs
        public partial class Startup
        {
        }
        ```

    - 加入 Configuration 方法

        ```cs
        public void Configuration(IAppBuilder app)
        {
            ConfigureAuth(app);
        }
        ```

    - 在 namespace 上加入 OwinStartupAttribute

        ```cs
        [assembly: OwinStartup(typeof(IdentityMongo.Startup))]
        ```

    - 完整程式碼

        ```cs
        using Microsoft.Owin;
        using Owin;

        [assembly: OwinStartup(typeof(IdentityMongo.Startup))]
        namespace IdentityMongo
        {
            public partial class Startup
            {
                public void Configuration(IAppBuilder app)
                {
                    ConfigureAuth(app);
                }
            }
        }
        ```

## 實際使用

1. 建立 `AccountController` 繼承 Controller

    ```cs
    public class AccountController : Controller
    {

    }
    ```

2. 加入 `SignInManager` 與 `UserManager`

    ```cs
    private ApplicationSignInManager _signInManager;
    private ApplicationUserManager _userManager;
    public ApplicationSignInManager SignInManager
    {
        get => _signInManager ?? HttpContext.GetOwinContext().Get<ApplicationSignInManager>();
        private set => _signInManager = value;
    }

    public ApplicationUserManager UserManager
    {
        get => _userManager ?? HttpContext.GetOwinContext().GetUserManager<ApplicationUserManager>();
        private set => _userManager = value;
    }
    ```

3. 加入建構子

    ```cs
    public AccountController()
    {
    }

    public AccountController(ApplicationUserManager userManager, ApplicationSignInManager signInManager)
    {
        UserManager = userManager;
        SignInManager = signInManager;
    }
    ```

4. 加入 Register 功能
    - AccountController

        ```cs
        [AllowAnonymous]
        public ActionResult Register()
        {
            return View();
        }

        //
        // POST: /Account/Register
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> Register(RegisterViewModel model)
        {
            if (ModelState.IsValid)
            {
                var user = new ApplicationUser { UserName = model.Email, Email = model.Email };
                var result = await UserManager.CreateAsync(user, model.Password);
                if (result.Succeeded)
                {
                    await SignInManager.SignInAsync(user, isPersistent: false, rememberBrowser: false);
                    return RedirectToAction("Index", "Home");
                }
                AddErrors(result);
            }
            return View(model);
        }
        private void AddErrors(IdentityResult result)
        {
            foreach (var error in result.Errors)
            {
                ModelState.AddModelError("", error);
            }
        }
        ```

    - 建立 `RegisterViewModel`

        ```cs
        public class RegisterViewModel
        {
            [Required]
            [EmailAddress]
            [Display(Name = "Email")]
            public string Email { get; set; }

            [Required]
            [StringLength(100, ErrorMessage = "The {0} must be at least {2} characters long.", MinimumLength = 6)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }

            [DataType(DataType.Password)]
            [Display(Name = "Confirm password")]
            [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
            public string ConfirmPassword { get; set; }
        }
        ```

    - 建立 View

        ```cs
        @model AddIdentity.Models.RegisterViewModel
        @{
            ViewBag.Title = "Register";
        }

        <h2>@ViewBag.Title.</h2>

        @using (Html.BeginForm("Register", "Account", FormMethod.Post, new { @class = "form-horizontal", role = "form" }))
        {
            @Html.AntiForgeryToken()
            <h4>Create a new account.</h4>
            <hr />
            @Html.ValidationSummary("", new { @class = "text-danger" })
            <div class="form-group">
                @Html.LabelFor(m => m.Email, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.TextBoxFor(m => m.Email, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(m => m.Password, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.PasswordFor(m => m.Password, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(m => m.ConfirmPassword, new { @class = "col-md-2 control-label" })
                <div class="col-md-10">
                    @Html.PasswordFor(m => m.ConfirmPassword, new { @class = "form-control" })
                </div>
            </div>
            <div class="form-group">
                <div class="col-md-offset-2 col-md-10">
                    <input type="submit" class="btn btn-default" value="Register" />
                </div>
            </div>
        }
        ```

5. 實際效果

    ![1result](https://user-images.githubusercontent.com/3851540/43404828-04b854a2-944b-11e8-8776-c3bfce25b86f.png)

## 心得

其實我覺得應該沒什麼機會再使用 MongoDB 來儲存 ASP.NET Identity 資訊，更不用說從無至有透過 Empty project template 來加入基於 MongoDB 的 ASP.NET Identity 功能，同時也認為這份的筆記對其他人幫助有限，但經過了幾次實際經驗：在開發當下覺得沒什麼難度的設定或是功能，常常日後一直回想不起當時的做法與思維，與其未來花更多時間來試想，不如趁著記憶猶新時順手做個筆記，不一定為了將來需要，也不一定為了幫助誰，就當做為某個專案功能留個紀念也好

[專案原始碼](https://github.com/yowko/ASP.NET-Identity_MongoDB-Sample.git)

## 參考資訊

1. [使用 MongoDB 儲存 ASP.NET Identity 資料](/2018/07/aspnet-identity-mongodb.html)
2. [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/2017/11/add-aspnet-identity-empty-project.html)
3. [專案原始碼](https://github.com/yowko/ASP.NET-Identity_MongoDB-Sample.git)
