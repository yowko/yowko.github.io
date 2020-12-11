---
title: "ASP.NET Identity 如何依據不同用途設定個別 Token 時效"
date: 2018-04-03T01:04:00+08:00
lastmod: 2020-12-11T01:04:26+08:00
draft: false
tags: ["ASP.NET Identity"]
slug: "multiple-tokenlifespan"
aliases:
    - /2018/04/multiple-tokenlifespan.html
---
# ASP.NET Identity 如何依據不同用途設定個別 Token 時效
之前曾經筆記中 [改 ASP.NET Identity 2 的 Token 時效](/2017/12/aspnet-identity-2-token-lifetime.html) 紀錄到 ASP.NET Identity 預設的 token 時效及調整方式。在專案實際使用時，user 提出其他需求：依不同功能別而有不同的 token 時效，例：註冊新帳號 - 確認 E-mail 的時效為 24 小時；忘記密碼 - 重設密碼的時效為 15 分鐘

針對這個需求，有好幾個可行方案閃過，但沒有實際驗證過誰也說不準能不能用，所以立馬來試試看囉

## Solution 1：建立不同的 UserManager

1.  IdentityConfig.cs 新增 UserManager


    *   新增內容 (與預設的 ApplicationUserManager 同級)

        ```cs
        public class MailUserManager : UserManager<ApplicationUser>
        {
            public MailUserManager(IUserStore<ApplicationUser> store) : base(store)
            {
            }
                        
            public static MailUserManager Create(IdentityFactoryOptions<MailUserManager> options, IOwinContext context)
            {
                var manager = new MailUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));
                manager.EmailService = new EmailService();
                var dataProtectionProvider = options.DataProtectionProvider;
                if (dataProtectionProvider != null)
                {
                    manager.UserTokenProvider =
                        new DataProtectorTokenProvider<ApplicationUser>(dataProtectionProvider.Create("ASP.NET Identity"))
                        {
                            TokenLifespan = TimeSpan.FromMinutes(15)
                        };
                }
                return manager;
            }
        }
        ```

    *   完整檔案內容

        ```cs
        using System;
        using System.Security.Claims;
        using System.Threading.Tasks;
        using Microsoft.AspNet.Identity;
        using Microsoft.AspNet.Identity.EntityFramework;
        using Microsoft.AspNet.Identity.Owin;
        using Microsoft.Owin;
        using Microsoft.Owin.Security;
        using DemoIdentity.Models;
        
        namespace DemoIdentity
        {
            public class EmailService : IIdentityMessageService
            {
                public Task SendAsync(IdentityMessage message)
                {
                    // Plug in your email service here to send an email.
                    System.Net.Mail.SmtpClient MySmtp = new System.Net.Mail.SmtpClient("127.0.0.1", 25);
                    return MySmtp.SendMailAsync("service@yowko.com", message.Destination, message.Subject, message.Body);
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
                public ApplicationUserManager(IUserStore<ApplicationUser> store) : base(store)
                {
                }
                
                public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context) 
                {
                    var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));
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
                        RequireNonLetterOrDigit = false,
                        RequireDigit = false,
                        RequireLowercase = false,
                        RequireUppercase = false,
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
            
            public class MailUserManager : UserManager<ApplicationUser>
            {
                public MailUserManager(IUserStore<ApplicationUser> store) : base(store)
                {
                }
                            
                public static MailUserManager Create(IdentityFactoryOptions<MailUserManager> options, IOwinContext context)
                {
                    var manager = new MailUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));
                    manager.EmailService = new EmailService();
                    var dataProtectionProvider = options.DataProtectionProvider;
                    if (dataProtectionProvider != null)
                    {
                        manager.UserTokenProvider =
                            new DataProtectorTokenProvider<ApplicationUser>(dataProtectionProvider.Create("ASP.NET Identity"))
                            {
                                TokenLifespan = TimeSpan.FromMinutes(15)
                            };
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

2.  Startup.Auth.cs 設定建立上述新增的 UserManager
    *   新增內容 (新增至 `ConfigureAuth` 方法中)

        ```cs
        app.CreatePerOwinContext<MailUserManager>(MailUserManager.Create);
        ```


    *   完整檔案內容

        ```cs
        using System;
        using Microsoft.AspNet.Identity;
        using Microsoft.AspNet.Identity.Owin;
        using Microsoft.Owin;
        using Microsoft.Owin.Security.Cookies;
        using Owin;
        using DemoIdentity.Models;
        
        namespace DemoIdentity
        {
            public partial class Startup
            {
                // For more information on configuring authentication, please visit https://go.microsoft.com/fwlink/?LinkId=301864
                public void ConfigureAuth(IAppBuilder app)
                {
                    // Configure the db context, user manager and signin manager to use a single instance per request
                    app.CreatePerOwinContext(ApplicationDbContext.Create);
                    app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
                    app.CreatePerOwinContext<MailUserManager>(MailUserManager.Create);
                    app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);
                    
                    // Enable the application to use a cookie to store information for the signed in user
                    // and to use a cookie to temporarily store information about a user logging in with a third party login provider
                    // Configure the sign in cookie
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
                        }
                    });            
                    app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);
                    
                    // Enables the application to temporarily store user information when they are verifying the second factor in the two-factor authentication process.
                    app.UseTwoFactorSignInCookie(DefaultAuthenticationTypes.TwoFactorCookie, TimeSpan.FromMinutes(5));
                    
                    // Enables the application to remember the second login verification factor such as phone or email.
                    // Once you check this option, your second step of verification during the login process will be remembered on the device where you logged in from.
                    // This is similar to the RememberMe option when you log in.
                    app.UseTwoFactorRememberBrowserCookie(DefaultAuthenticationTypes.TwoFactorRememberBrowserCookie);
                    
                    // Uncomment the following lines to enable logging in with third party login providers
                    //app.UseMicrosoftAccountAuthentication(
                    //    clientId: "",
                    //    clientSecret: "");
                                //app.UseTwitterAuthentication(
                    //   consumerKey: "",
                    //   consumerSecret: "");
                                //app.UseFacebookAuthentication(
                    //   appId: "",
                    //   appSecret: "");
                                //app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
                    //{
                    //    ClientId = "",
                    //    ClientSecret = ""
                    //});
                }
            }
        }
        ```

3.  在 controller 中建立新增的 UserManager
    *   新增內容

        ```cs
        private MailUserManager _mailUserManager;
        public MailUserManager MailUserManager
        {
            get
            {
                return _mailUserManager ?? HttpContext.GetOwinContext().GetUserManager<MailUserManager>();
            }
            private set
            {
                _mailUserManager = value;
            }
        }
        public AccountController(MailUserManager mailUserManager, ApplicationUserManager userManager, ApplicationSignInManager signInManager)
        {
            UserManager = userManager;
            SignInManager = signInManager;
            MailUserManager = mailUserManager;
        }
        ```

4.  使用新增的 UserManager 做為產生 token 的物件
    *   以 `Register` 為例

        ```cs
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
                    await SignInManager.SignInAsync(user, isPersistent:false, rememberBrowser:false);
                    
                    // For more information on how to enable account confirmation and password reset please visit https://go.microsoft.com/fwlink/?LinkID=320771
                    // Send an email with this link
                    string code = await MailUserManager.GenerateEmailConfirmationTokenAsync(user.Id);
                    var callbackUrl = Url.Action("ConfirmEmail", "Account", new { userId = user.Id, code = code }, protocol: Request.Url.Scheme);
                    await MailUserManager.SendEmailAsync(user.Id, "Confirm your account", "Please confirm your account by clicking <a href=\"" + callbackUrl + "\">here</a>");
                    
                    return RedirectToAction("Index", "Home");
                }
                AddErrors(result);
            }
            
            // If we got this far, something failed, redisplay form
            return View(model);
        }
        ```

    *   對應的 `ConfirmEmail` 也需調整

        ```cs
        [AllowAnonymous]
        public async Task<ActionResult> ConfirmEmail(string userId, string code)
        {
            if (userId == null || code == null)
            {
                return View("Error");
            }
            var result = await MailUserManager.ConfirmEmailAsync(userId, code);
            return View(result.Succeeded ? "ConfirmEmail" : "Error");
        }
        ```

## Solution 2：擴充 UserManager 方法

1.  IdentityConfig.cs 加入不同 `DataProtectorTokenProvider` 並新增自訂方法用來存取 token
    *   新增內容至 `ApplicationUserManager` class 中

        ```cs
        private static Microsoft.AspNet.Identity.Owin.DataProtectorTokenProvider<ApplicationUser> tokenprovider;
        /// <summary>Get a user token for a specific purpose</summary>
        /// <param name="purpose"></param>
        /// <param name="userId"></param>
        /// <returns></returns>
        public virtual async Task<string> GenerateUserResetTokenAsync(string purpose, string userId)
        {
            ApplicationUser user = await FindByIdAsync(userId).WithCurrentCulture<ApplicationUser>();
            if (user==null)
            {
                return string.Empty;
            }
            return await tokenprovider.GenerateAsync(purpose, this, user).WithCurrentCulture<string>();
        }
        /// <summary>Get a user token for a specific purpose</summary>
        /// <param name="purpose"></param>
        /// <param name="userId"></param>
        /// <returns></returns>
        public virtual async Task<bool> VerifyEmailResetTokenAsync(string purpose, string userId, string token)
        {
            ApplicationUser user = await this.FindByIdAsync(userId).WithCurrentCulture<ApplicationUser>();
            if (user == null)
            {
                return false;
            }
            return await tokenprovider.ValidateAsync(purpose, token, this, user).WithCurrentCulture<bool>();
        }
        ```

    *   修改 `Create` 方法，加入以下程式碼

        ```cs
        tokenprovider = new Microsoft.AspNet.Identity.Owin.DataProtectorTokenProvider<ApplicationUser>(options.DataProtectionProvider.Create("ASP.NET Identity"))
        {
            TokenLifespan = TimeSpan.FromMinutes(5)
        };
        ```
    *   完整檔案內容

        ```cs
        using System;
        using System.Data.Entity.Utilities;
        using System.Security.Claims;
        using System.Threading.Tasks;
        using Microsoft.AspNet.Identity;
        using Microsoft.AspNet.Identity.EntityFramework;
        using Microsoft.AspNet.Identity.Owin;
        using Microsoft.Owin;
        using Microsoft.Owin.Security;
        using DemoIdentity.Models;
        
        namespace DemoIdentity
        {
            public class EmailService : IIdentityMessageService
            {
                public Task SendAsync(IdentityMessage message)
                {
                    // Plug in your email service here to send an email.
                    System.Net.Mail.SmtpClient MySmtp = new System.Net.Mail.SmtpClient("127.0.0.1", 25);
                    return MySmtp.SendMailAsync("service@yowko.com", message.Destination, message.Subject, message.Body);
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
                private static Microsoft.AspNet.Identity.Owin.DataProtectorTokenProvider<ApplicationUser> tokenprovider;
                /// <summary>Get a user token for a specific purpose</summary>
                /// <param name="purpose"></param>
                /// <param name="userId"></param>
                /// <returns></returns>
                public virtual async Task<string> GenerateUserResetTokenAsync(string purpose, string userId)
                {
                    ApplicationUser user = await FindByIdAsync(userId).WithCurrentCulture<ApplicationUser>();
                    if (user==null)
                    {
                        return string.Empty;
                    }
                    return await tokenprovider.GenerateAsync(purpose, this, user).WithCurrentCulture<string>();
                }
                /// <summary>Get a user token for a specific purpose</summary>
                /// <param name="purpose"></param>
                /// <param name="userId"></param>
                /// <returns></returns>
                public virtual async Task<bool> VerifyEmailResetTokenAsync(string purpose, string userId, string token)
                {
                    ApplicationUser user = await this.FindByIdAsync(userId).WithCurrentCulture<ApplicationUser>();
                    if (user == null)
                    {
                        return false;
                    }
                    return await tokenprovider.ValidateAsync(purpose, token, this, user).WithCurrentCulture<bool>();
                }
                public ApplicationUserManager(IUserStore<ApplicationUser> store) : base(store)
                {
                }
                
                public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context) 
                {
                    var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));
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
                        RequireNonLetterOrDigit = false,
                        RequireDigit = false,
                        RequireLowercase = false,
                        RequireUppercase = false,
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
                        tokenprovider = new Microsoft.AspNet.Identity.Owin.DataProtectorTokenProvider<ApplicationUser>(options.DataProtectionProvider.Create("ASP.NET Identity"))
                        {
                            TokenLifespan = TimeSpan.FromMinutes(1)
                        };
                    }
                    return manager;
                }
            }
            
            // Configure the application sign-in manager which is used in this application.
            public class ApplicationSignInManager : SignInManager<ApplicationUser, string>
            {
                public ApplicationSignInManager(ApplicationUserManager userManager, IAuthenticationManager authenticationManager) : base(userManager, authenticationManager)
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

2.  調整 AccountController
    *   以 ForgotPassword 為例

        ```cs
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> ForgotPassword(ForgotPasswordViewModel model)
        {
            if (ModelState.IsValid)
            {
                var user = await UserManager.FindByNameAsync(model.Email);
                if (user == null || !(await UserManager.IsEmailConfirmedAsync(user.Id)))
                {
                    // Don't reveal that the user does not exist or is not confirmed
                    return View("ForgotPasswordConfirmation");
                }
                
                // For more information on how to enable account confirmation and password reset please visit https://go.microsoft.com/fwlink/?LinkID=320771
                // Send an email with this link
                string code = await UserManager.GenerateUserResetTokenAsync("ResetPassword", user.Id);
                var callbackUrl = Url.Action("ResetPassword", "Account", new { userId = user.Id, code = code }, protocol: Request.Url.Scheme);
                await UserManager.SendEmailAsync(user.Id, "Reset Password", "Please reset your password by clicking <a href=\"" + callbackUrl + "\">here</a>");
                return RedirectToAction("ForgotPasswordConfirmation", "Account");
            }
            
            // If we got this far, something failed, redisplay form
            return View(model);
        }
        ```

    *   實際 ResetPassword 也需要調整

        ```cs
        [AllowAnonymous]
        public ActionResult ResetPassword(string code, string userId)
        {
            if (string.IsNullOrWhiteSpace(code) || string.IsNullOrWhiteSpace(userId) || !UserManager.VerifyEmailResetTokenAsync("ResetPassword", userId, code).Result)
            {
                return View("Error");
            }
            return View();
        }
        ```

## Solution 3：自訂 DataProtectorTokenProvider

1.  新增 `DataProtectorTokenProviderEx.cs`

    ```cs
    using System;
    using System.Data.Entity.Utilities;
    using System.Globalization;
    using System.IO;
    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.AspNet.Identity;
    using Microsoft.Owin.Security.DataProtection;
    
    namespace DemoIdentity.Extension
    {
        /// <summary>
        /// Token provider that uses an IDataProtector to generate encrypted tokens based off of the security stamp
        /// </summary>
        public class DataProtectorTokenProviderEx<TUser, TKey> : IUserTokenProvider<TUser, TKey>
            where TUser : class, IUser<TKey>
            where TKey : IEquatable<TKey>
        {
            /// <summary>
            /// Constructor
            /// </summary>
            public DataProtectorTokenProviderEx(IDataProtector protector)
            {
                if (protector == null)
                {
                    throw new ArgumentNullException("protector");
                }
                
                Protector = protector;
                TokenLifespan = TimeSpan.FromDays(1.0);
            }
            
            /// <summary>
            /// IDataProtector for the token
            /// </summary>
            public IDataProtector Protector { get; private set; }
            
            /// <summary>
            /// Lifespan after which the token is considered expired
            /// </summary>
            public TimeSpan TokenLifespan { get; set; }
            
            /// <summary>
            /// Generate a protected string for a user
            /// </summary>
            public async Task<string> GenerateAsync(string purpose, UserManager<TUser, TKey> manager, TUser user)
            {
                if (user == null)
                {
                    throw new ArgumentNullException("user");
                }
                
                var memoryStream = new MemoryStream();
                using (var binaryWriter = new BinaryWriter(memoryStream, new UTF8Encoding(false, true), true))
                {
                    binaryWriter.Write(DateTimeOffset.UtcNow.UtcTicks);
                    binaryWriter.Write(Convert.ToString(user.Id, CultureInfo.InvariantCulture));
                    binaryWriter.Write(purpose ?? "");
                    string stamp = null;
                    if (manager.SupportsUserSecurityStamp)
                    {
                        stamp = await manager.GetSecurityStampAsync(user.Id);
                    }
                    
                    binaryWriter.Write(stamp ?? "");
                }
                
                byte[] token = this.Protector.Protect(memoryStream.ToArray());
                return Convert.ToBase64String(token);
            }
            
            /// <summary>
            /// Return false if the token is not valid
            /// </summary>
            public async Task<bool> ValidateAsync(string purpose, string token, UserManager<TUser, TKey> manager, TUser user)
            {
                try
                {
                    var unprotectedData = Protector.Unprotect(Convert.FromBase64String(token));
                    var ms = new MemoryStream(unprotectedData);
                    
                    using (var reader = new BinaryReader(ms, new UTF8Encoding(false, true), true))
                    {
                        var creationTime = new DateTimeOffset(reader.ReadInt64(), TimeSpan.Zero);
                        var tokenLifespan = TokenLifespan;
                        //加上如果特定目的就使用不同時效
                        if (purpose == "ResetPassword")
                        {
                            tokenLifespan = TimeSpan.FromMinutes(15);
                        }
                        
                        var expirationTime = creationTime + tokenLifespan;
                        if (expirationTime < DateTimeOffset.UtcNow)
                        {
                            return false;
                        }
                        
                        var userId = reader.ReadString();
                        if (!String.Equals(userId, Convert.ToString(user.Id, CultureInfo.InvariantCulture)))
                        {
                            return false;
                        }
                        
                        var purp = reader.ReadString();
                        if (!String.Equals(purp, purpose))
                        {
                            return false;
                        }
                        
                        var stamp = reader.ReadString();
                        if (reader.PeekChar() != -1)
                        {
                            return false;
                        }
                        
                        if (manager.SupportsUserSecurityStamp)
                        {
                            var expectedStamp = await manager.GetSecurityStampAsync(user.Id).WithCurrentCulture();
                            return stamp == expectedStamp;
                        }
                        
                        return stamp == "";
                    }
                }
                catch
                {
                    return false;
                }
            }
            
            /// <summary>
            /// Returns true if the provider can be used to generate tokens for this user
            /// </summary>
            public Task<bool> IsValidProviderForUserAsync(UserManager<TUser, TKey> manager, TUser user)
            {
                return Task.FromResult(true);
            }
            
            /// <summary>
            /// This provider no-ops by default when asked to notify a user
            /// </summary>
            public Task NotifyAsync(string token, UserManager<TUser, TKey> manager, TUser user)
            {
                return Task.FromResult(0);
            }
        }
    }
    ```

2.  IdentityConfig.cs 使用自訂 DataProtectorTokenProvider - `DataProtectorTokenProviderEx`

    ```cs
    if (dataProtectionProvider != null)
    {
        manager.UserTokenProvider = 
            new DataProtectorTokenProviderEx<ApplicationUser,string>(dataProtectionProvider.Create("ASP.NET Identity"));
    }
    ```

3.  AccountController 使用無需任何調整


## 心得

上述三種方式都可以達到針對不同目的設定個別的 token 時效，只是選擇上各有優缺點

1.  Solution 1：建立不同的 UserManager
    *   優點：最直覺
    *   缺點：相同的 code 太多

2.  Solution 2：擴充 UserManager 方法
    *   優點：Identity 底層升級不易受到影響
    *   缺點：使用上容易出現混淆

3.  Solution 3：自訂 DataProtectorTokenProvider
    *   優點：使用角度完全不需調整
    *   缺點：可能會與 Identity 核心脫鉤



第一種做法冗贅的 code 太多直接排除，而在 `擴充 UserManager` 與 `自訂 DataProtectorTokenProvider` 間，我個人選擇 `擴充 UserManager`，原因是讓專案變動的範圍限縮，UserManager 跟 AccountController 都是專案內容的一部份，本來就會異動及修改，而不另外自訂 `DataProtectorTokenProvider`，避免日後如果遇到需要 update Identity 版本時，得手動將異動內容套用到自訂的 DataProtectorTokenProvider 上，也可以降低團隊成員額外理解 `DataProtectorTokenProvider` 的難度

# 參考資訊

1.  [ArtemAvramenko/DataProtectorTokenProviderEx.cs](https://gist.github.com/ArtemAvramenko/8f34ec8aac5c0fb6380871a18e69ffd8#file-dataprotectortokenproviderex-cs)
2.  [AspNetIdentity/src/Microsoft.AspNet.Identity.Owin/DataProtectorTokenProvider.cs](https://github.com/aspnet/AspNetIdentity/blob/9c48993a446288032f9824633e6dae81257da06e/src/Microsoft.AspNet.Identity.Owin/DataProtectorTokenProvider.cs)
3.  [改 ASP.NET Identity 2 的 Token 時效](/2017/12/aspnet-identity-2-token-lifetime.html)
