---
title: "讓 .NET Core 的 HttpClientFactory 不驗證 Https 憑證"
date: 2019-03-06T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["csharp","dotnet core","Docker"]
slug: "dotnet-core-httpclientfactory-invalid-certificate"
---
## 讓 .NET Core 的 HttpClientFactory 不驗證 Https 憑證

Https 幾乎已成為了現在網站的基本配備，從過去只有敏感交易網站才需要，到現在瀏覽器還會把非 Https 網站標記為 `不安全`，而 .NET Core 程式在預設專案範本下也會啟用 Https，不過因為開發環境的 SSL 憑證只是開發測試用，沒有實際效力並不被信任，因此透過 HttpClient call 時就會出現憑證無效的錯誤，但在正式環境上就會綁定正確憑證，所以得要在 Development 環境避免驗證憑證有效性，而在正式機上就需要啟用驗證功能，來看看如何設定吧

## 基本環境說明

1. .NET Core 2.2.101
2. 原始程式碼

    ```cs
    services.AddHttpClient("yowkoblog", c =>
    {
        c.BaseAddress = new Uri("/");
    });
    ```

    > .NET Core 使用 HttpClientFactory 完整設定方式請參考 [在 .NET Core 與 .NET Framework 上使用 HttpClientFactory](/httpclientfactory-dotnet-core-dotnet-framework/#%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F)

## 錯誤訊息

- 訊息內容

    ```log
    System.Net.Http.HttpRequestException: The SSL connection could not be established, see inner exception. ---> System.Security.Authentication.AuthenticationException: The remote certificate is invalid according to the validation procedure.
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/53889137-d967b000-4060-11e9-8aba-5874a4fd49fe.png)

## 修改方式 (擇一即可)

1. 直接回傳 true

    ```cs
    services.AddHttpClient("yowkoblog", c =>
    {
        c.BaseAddress = new Uri("/");
    }).ConfigurePrimaryHttpMessageHandler(h =>
    {
        var handler = new HttpClientHandler();
        if (_env_.IsDevelopment())
        {
            handler.ServerCertificateCustomValidationCallback = delegate { return true; };
        }
        return handler;
    });
    ```

2. 使用 HttpClientHandler 屬性

    ```cs
    services.AddHttpClient("yowkoblog", c =>
    {
        c.BaseAddress = new Uri("/");
    }).ConfigurePrimaryHttpMessageHandler(h =>
    {
        var handler = new HttpClientHandler();
        if (_env_.IsDevelopment())
        {
            handler.ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;
        }
        return handler;
    });
    ```

## 實際使用

```cs
[ApiController]
[Route("[controller]")]
public class AccountsController : ControllerBase
{
    private readonly IHttpClientFactory _blogClientFactory;
    public BlogController(IHttpClientFactory blogClientFactory)
    {
        _blogClientFactory = blogClientFactory;
    }
    [HttpPost("GetPosts")]
    public async Task<ActionResult<string>> GetPosts()
    {
       //名稱與 services.AddHttpClient 註冊時相同
        var client = _platformAuthFactory.CreateClient("yowkoblog");

        var response = await (await client.PostAsync(authPath, null)).Content.ReadAsStringAsync();

        return response;
    }
}
```

## 完整程式碼

1. Startup.cs

    ```cs
    public class Startup
    {
        private IHostingEnvironment currentEnvironment { get; set; }
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.Configure<CookiePolicyOptions>(options =>
            {
                // This lambda determines whether user consent for non-essential cookies is needed for a given request.
                options.CheckConsentNeeded = context => true;
                options.MinimumSameSitePolicy = SameSiteMode.None;
            });

            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
            services.AddHttpClient("yowkoblog", c => { c.BaseAddress = new Uri("https://localhost:5000/"); })
                .ConfigurePrimaryHttpMessageHandler(h =>
                {
                    var handler = new HttpClientHandler();
                    if (currentEnvironment.IsDevelopment())
                    {
                        //handler.ServerCertificateCustomValidationCallback = delegate { return true; };
                        handler.ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;
                    }
                    return handler;
                });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            currentEnvironment = env;
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseStaticFiles();
            app.UseCookiePolicy();

            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
    ```

2. Controller

    ```cs
    [ApiController]
    [Route("[controller]")]
    public class AccountsController : ControllerBase
    {
        private readonly IHttpClientFactory _blogClientFactory;
        public BlogController(IHttpClientFactory blogClientFactory)
        {
            _blogClientFactory = blogClientFactory;
        }
        [HttpPost("GetPosts")]
        public async Task<ActionResult<string>> GetPosts()
        {
        //名稱與 services.AddHttpClient 註冊時相同
            var client = _platformAuthFactory.CreateClient("yowkoblog");

            var response = await (await client.PostAsync(authPath, null)).Content.ReadAsStringAsync();

            return response;
        }
    }
    ```

## 心得

這個問題好像曾經遇過，但當時似乎是覺得這個小小設定實在不需要紀錄，想不到再次相見又花了我二十分鐘 XD

但過去我只知道在 certificate callback 永遠回傳 true 來處理，這次重新 review 發現透過使用 `HttpClientHandler.DangerousAcceptAnyServerCertificateValidator` 語意更清楚，.NET Core 真是令人驚豔呀

## 參考資訊

1. [bypass invalid SSL certificate in .net core](https://stackoverflow.com/a/44540071)
2. [ASP Core HttpClientFactory Pattern Use Client Cert](https://stackoverflow.com/a/52372961)
3. [Add DangerousAcceptAnyServerCertificateValidator property to HttpClient](https://github.com/dotnet/corefx/pull/19908)
